---
layout: post
title:  "Fast SFTP transferts using PHP"
date:   2016-01-20
categories: workflow
tags: [php, sftp, rasmus, phpseclib, php-ssh2]
author: romain
---

Recently, I've faced a tricky problem at Etsy France that implied two implementations that gave counter-intuitive performances.

Here is how it started: we needed to daily upload and download CSV files from our email provider and these transfers had to be using the [SFTP protocol](https://en.wikipedia.org/wiki/SSH_File_Transfer_Protocol).

While I was looking how it was possible to implement SFTP transfers inside a PHP cron task,
I remembered the existence of a PHP extension - [PHP-SSH2](http://php.net/manual/en/book.ssh2.php),
which was based on the [libssh2 C binding](http://www.libssh2.org/), and wrote the following code for a POC:

{% highlight php %}
<?php
// sftp_ssh2.php

include 'config.php'; // contains $host, $user, $password, $localCsvFile and $destinationCsvFile

$connection = ssh2_connect($host, 22);

if (!is_resource($connection)) {
    throw new \RunTimeException(sprintf(
        'Cannot connect on remote server %s on port 22',
        $host
    ));
}

ssh2_auth_password($connection, $user, $password);

// Initialize SFTP Subsystem to send the file
$sftp = ssh2_sftp($connection);

// Open both local and remote streams
// Note : ssh2.sftp:// wrapper is provided by the PHP-SSH2 extension
$src  = fopen($localCsvFile, 'r');
$dest = fopen("ssh2.sftp://{$sftp}/{$destinationCsvFile}", 'w');

// Let's copy !
stream_copy_to_stream($src, $dest))

// Close streams
fclose($src);
fclose($dest);

{% endhighlight %}

I've tested this code with a file of ~10MB between my local machine and the remote server, and quickly found that the transfer - over a fiber connection - appeared to a bit sluggish : 20 seconds.

To confirm that intuition, I ran the same transfer again checking if we had a network congestion issue or using the `time` utility trying to figure out what was the cause of that duration.

{% highlight bash %}
$ /usr/local/bin/time -p php sftp_ssh2.php
real 20.72
user 0.16
sys 0.16
{% endhighlight %}

I finally compared it to one done with [FileZilla](https://filezilla-project.org/) and it was completed in less than 2 seconds.

I did another test by launching the same script on the EC2 instance where the upload was planned to be run
in production but it gave substantially the same difference, between 9 and 10 times slower than
the `sftp` command with ~10Mb File.

The transfer with the PHP script was about 10 times slower than ones with other SFTP clients in the
same conditions (same destination server, same Internet connection).

That performance was really unexpected since the extension is supposed to be a simple binding over a C implementation,
and even if it has to convert PHP data structure to ones usable by libssh2, the overall duration seemed way too high.

I could live with it, but since I'm a curious person, I took the opportunity to ask our [smart friends at Etsy US](https://codeascraft.com/)
about the problem and [Rasmus Lerdorf](https://en.wikipedia.org/wiki/Rasmus_Lerdorf) came to our rescue !

#### Enter Sync / Async

After investigations, it appeared that libssh2 only supports synchronous transfers
whereas command-line `sftp` does asynchronous ones.

Synchronous transfer means that each SFTP packet (defined to 32KB by SFTP protocol) sent over
the network will have to wait for an acknowledge message from the server, as explained by
Daniel Stenberg (author of curl and libssh) [here](http://daniel.haxx.se/blog/2010/12/08/making-sftp-transfers-fast/).
So it slows things down a lot because it has to wait for an ACK on each packet sent rather than sending data in bulk.


#### Solutions

Our first possible solution was to launch a basic [exec](http://php.net/manual/fr/function.exec.php) from PHP to launch transfert via `sftp` command.
It felt inelegant to fork and authentication using keys without passphrase would have been mandatory.

More research showed another way to go with the [phpseclib](http://phpseclib.sourceforge.net/), which is a pure PHP reimplementation
of some secure protocols, including SFTP.
I've setup the library locally and I've launched the same test as with PHP-SSH2 extension :


{% highlight php %}
<?php
// sftp_phpseclib.php

include 'config.php'; // contains $host, $user, $password, $localCsvFile and $destinationCsvFile
require_once 'vendor/autoload.php';

use phpseclib\Net\SFTP;

$sftp = new SFTP($host, 22);

if (!$sftp->login($user, $password)) {
    throw new \RunTimeException(sprintf(
        'Cannot connect on remote server %s on port 22',
        $host
    ));
}

$sftp->put($destinationCsvFile, $localCsvFile, SFTP::SOURCE_LOCAL_FILE);

{% endhighlight %}

*Note : the code is also cleaner and more readable than with PHP-SSH2, since the library provides a nice object oriented API*

Let's test it :

{% highlight bash %}
$ /usr/local/bin/time -p php sftp_phpseclib.php
real 1.42
user 0.26
sys 0.05
{% endhighlight %}

Wow, that's nearly the same performance than the `sftp` command would have gave !


## TODO Finish conclusion
