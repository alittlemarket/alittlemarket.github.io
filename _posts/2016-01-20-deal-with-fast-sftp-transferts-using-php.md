---
layout: post
title:  "Deal with fast SFTP transferts using PHP"
date:   2016-01-20
categories: workflow
tags: [php, sftp, rasmus, phpseclib, php-ssh2]
author: romain
---

Recently, I've faced a tricky problem at Etsy France.
The project was to implement some CSV file transfers of useful data between us and our email provider (both upload and download).
These transfers were performed by a cron task each day and the protocol used for these transfers was SFTP.

So I've searched how it was possible to implement SFTP transfers inside a PHP cron task, and I remember the [PHP-SSH2](http://php.net/manual/en/book.ssh2.php) extension, which provides a useful API to implements that.

So, let's start the work with the help of the extension, here is the code i've implemented for a POC :

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

Then I've tested this code, with a file of ~10MB between my local machine and the remote server, and I quickly found that the transfer appears to be slow.

Indeed before running the script, I've tested the same transfer with [FileZilla](https://filezilla-project.org/) and with my current fiber connection,
FileZilla said the transfert was completed in less than 2 seconds.

I've ensured the transfert was slow with `time` utility :

{% highlight bash %}
$ /usr/local/bin/time -p php sftp_ssh2.php
real 20.72
user 0.16
sys 0.16
{% endhighlight %}

So, the transfert with the PHP script is 10 times slower than the transfert with FileZilla client in same conditions (same destination server, same internet connection).

I did another test by launching the same script on the EC2 instance where the script was planned to be runned once in production, and it was the same result, between 9 and 10 times slower than `sftp` command with ~10Mb File.

I know that using SFTP protocol from PHP with PHP-SSH2 extension can have overhead compared to pure `sftp` command, because the extension uses the [libssh2](http://www.libssh2.org/) underlying library.
So there are 3 layers :

- the PHP userland
- the PHP extension
- and finally libssh2

But 10 times slower is definitly too much. I could live with it, but as I'm a curious person and I've got the opportunity to ask to smart people about the problem like Rasmus, let's do it !

#### Enter Sync / Async

After investigations, it seems that the fact that libssh2 only supports synchronous transfers
whereas command-line `sftp` will do async.

Synchronous transfert means that each packet (defined to 32KB by SFTP protocol) transfered over the network will have to wait for an ACK from the server, as explained by
Daniel Stenberg (author of curl and libssh) [here](http://daniel.haxx.se/blog/2010/12/08/making-sftp-transfers-fast/).
So it slow things down a lot because it has to wait for an ACK on each packet sent.


#### Solution

The first possible solution was to launch a basic [exec](http://php.net/manual/fr/function.exec.php) from PHP to launch transfert via `sftp` command.

But before that, I've found [phpseclib](http://phpseclib.sourceforge.net/), which is a pure PHP reimplementation of some secure protocols, like SFTP !
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

*Note : the code is also cleaner and more readable than with PHP-SSH2, as the library provides a nice object oriented API*

Let's test it :

{% highlight bash %}
$ /usr/local/bin/time -p php sftp_phpseclib.php
real 1.42
user 0.26
sys 0.05
{% endhighlight %}


Wow, that's nearly same performance as `sftp` command line !


## TODO Finish conclusion
