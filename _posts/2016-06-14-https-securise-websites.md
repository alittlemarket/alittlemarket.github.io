---
layout: post
title:  "How Etsy France switched all its sites to HTTPS"
date:   2016-07-15
categories: projects
tags: [security, https, http, migration, project]
author: antoine
---

When Etsy acquired A little Market, the migration of our sites to https which was on the shelf since a few years became one of our top priority projects in mid 2015.  
But before starting such a big and impacting project, we went through a long preparation phase.

## Project preparation

### Why

Prevent session hijacking, data sniffed, etc.

### Goal

The goal of this project was to force all our users to access our services through HTTPS connections. Google and other search engines would have to crawl our websites only in HTTPS to avoid duplicate content.

### How

We want to switch smoothly and easily from HTTP to HTTPS. At most, we want to update a few configuration values to do it progressively.
All the blocking points to use HTTPS will have to be fixed before the deadline.

### Acceptance criteria

All our applications must be accessible through HTTPS.  
All users accessing the websites through HTTP must be redirected to HTTPS using 301 ([301 redirections](https://en.wikipedia.org/wiki/HTTP_301) are very important for Search engines).  

### When to switch ?

A switch from HTTP to HTTPS can have a big impact on search engine's ranking. As we do not use paid marketing, a bad indexing in search engines and especially Google can have really bad consequences on our traffic. Thanks to previous experiences, our SEO consultant estimated that SEO traffic returns back to normal around 3 months after switching.  
Because we had several marketing operations during the year (winter and summer sales, private sales, Christmas, etc.) and we didn't want a big impact on our SEO traffic during these operations, we carefully determined the periods when the switch could be made. We only found two:

- February/March (traffic should return back to normal around May)
- July/August (traffic should return back to normal around October)


## Milestones

1. Dependencies
    1. List all the dependencies of our main applications (ex: assets, blogs, images, etc)
    2. These dependencies will have to be migrated first.

2. We know some of our hostnames contain more than one subdomain level (ex: static.commun.alittlemarket.com) but we don't want to buy expensive certificates that cover every level.
    1. List all the concerned hostnames
    2. Fix incompatible subdomains (ex: static.commun.alittlemarket.com -> static-commun.alittlemarket.com). Fix them means:
        1. Create new subdomains in apache
        2. Add them in [Route53](https://aws.amazon.com/route53/)
        3. Update our stack to use them instead of the previous ones

3. Monitoring
   Monitor HTTP and HTTPS traffic using [Statsd](https://github.com/etsy/statsd)

4. Hardcoded URLs  
   Many links to our applications within our codebase are hardcoded
    1. Determine each type of hardcoded URL (ex: http://www.alittlemarket.com, http://CONSTANT, http://assets.alittlemarket.com, etc)
    2. Replace all the hardcoded URLs by the equivalent constant
        1. Project by project
        2. file type by file type (php, smarty, twig, etc.)
        3. URL type by URL type
        4. URLs in database

5. Google
    1. [Google Search Console](https://www.google.com/webmasters/tools/)  
       Create new entries for HTTPS sites
    2. [Google Analytics](https://analytics.google.com/)  
       Differentiate HTTP traffic from HTTPS traffic: we used Google Analytics' fifth custom var to store the scheme  
       `_gaq.push(['_setCustomVar', 5, 'Scheme', window.location.protocol.replace(/:$/, ''), 3]);`

6. Force HTTPS  
   Requests going through HTTP must be redirected to HTTPS  
   We don't want to block HTTP traffic

## Execution

We were two then three to work on this project during 4 months, between November 2015 and February 2016.  
It was a hard task as you can imagine due to the innumerable amount of hardcoded URLs everywhere in the codebase, the many dependencies that also needed to migrate to HTTPS, and the risk of losing SEO traffic.

During the project development, we never blocked the access to our sites through HTTPS but this protocol had a specific robots.txt file to prevent search engines from crawling.  
This played tricks on us on D-day...

```
User-agent: *
Disallow: /
```

But one important thing not to do is to disallow HTTP traffic from search engines once the switch is done.   
If you do, search engines can't calculate the new position of the HTTPS pages. **You must allow HTTP traffic and redirect it using a 301 to the HTTPS version**.

We also used a [HSTS header](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) for all incoming requests to warn web browsers that they should only interact with our applications using HTTPS protocol.

## Gradual activation

One of our goal was to progressively enable HTTPS on our applications, we didn't want to enable it for everybody at once.  
But on our websites, some of the oldest pages use the html tag `<base href="http://www.alittlemarket.com/" />` that we didn't succeed in removing without breaking many links everywhere. The link in this tag is managed by a configuration file and could not change based on the user accessing the page.

We decided to hack this feature to be able to adapt this link depending on the visited URL. In other words, if a user visits the website in HTTP, the link has to be http://www.alittlemarket.com; and if a user consults the website in HTTPS, the link has to be https://www.alittlemarket.com.
So, a user arriving in HTTPS on one of our websites stays in HTTPS during his entire session.

It looks like:

```php
<?php
// Load balancer define the key HTTP_X_REAL_PROTO
if (array_key_exists('HTTP_X_REAL_PROTO', $_SERVER)) {
    $scheme = $_SERVER['HTTP_X_REAL_PROTO'];
} else {
    $scheme = DEFAULT_HTTP_SCHEME;
}

if (!defined('HTTP_PATH_APP_DOMAIN_NAME')) {
    define('HTTP_PATH_APP_DOMAIN_NAME', sprintf('%s://%s', $scheme, HTTP_APP_DOMAIN_NAME));
}
```

### Activation on our preprod environment "Princess"

The first step was to enable HTTPS on our Princess environment.  
Everybody who wants to deploy a feature in production (several times a day) must test it first on Princess. So, everybody was testing HTTPS on Princess without thinking about it during several days.

Our team (Core team) and our QA team had also tested HTTPS during all the process.

### Activation on production (for admin only)

After 1 week of tests on Princess, we decided to enable the feature on production but just for the admins.  
So, when someone was detected as being an admin and was surfing on HTTP, he was automatically redirected to HTTPS by the application layer.

### Activation for everybody application by application

The big day was **February 29th**.  
The plan was to:

* Create a war room contaning one person of each service (customer service, communication, technical teams, QA, SEO members, etc.)
* Switch the applications one by one (alittlemarket.it, alittlemarket.com, alittlemercerie.com)
  * Open traffic through HTTPS
  * 301 redirections for incoming traffic though HTTP
  * Update all the sitemaps (a cronjob to relaunch) to use HTTPS URLs instead of HTTP
* After each switch, everybody in the war room had to check that everything was working correctly (metrics, functional tests, social networks, forums, etc.)
* Drink some beers after that big day !

### D-day traffic metrics

As you can see on the metrics below, we monitor global HTTP and HTTPS traffic, the traffic for each website, and Google bot traffic.

On the 3 graphs (ALM IT, ALM FR, and ALME FR) we saw that the HTTP traffic abruptly stops and at the same moment the HTTPS traffic starts. That's when we switched.  
On the global traffic we saw the same impact at the same times.  

Finally, we saw a strange thing on the Google bot traffic. At 2pm (14:00) we see that Google bot http traffic stops but the HTTPS traffic does not start. It's not normal and we were worried about that. We couldn't afford to lose SEO. After some research, we discovered that Google had cached the robots.txt file we served it when it came through HTTPS (it was a different file than the HTTPS robots.txt file). In that file, to avoid duplicated content, we disallowed everything which is why it stopped crawling them.  
Fortunately, we discovered that problem very quickly and 1.5 hours later, Google crawled our applications again. We only needed to send the good robots.txt in [Google Search Console](https://www.google.com/webmasters/tools/).

{:.text-center}
![Stasd metrics](/assets/https-securise-websites/grafana-traffic.png)

### Bugs

We found only one bug due to the switch.  
The bug was due to an external javascript library (used for the [Mondial Relay](http://www.mondialrelay.fr/) delivery service) that we still called using HTTP.  
Web browsers did not load the external javascript and the customers who chose Mondial Relay as delivery mode couldn't choose the relay point where they wanted to be delivered...  
To fix the bug we couldn't simply change the URL to use HTTPS because the library wasn't available through that protocol. We had to upgrade the version of the library. Fortunately, it only involved small changes and the bug was fixed quickly.

## What happened next?

### Google indexation

Thanks to 301 redirection from HTTP to HTTPS on every page of the sites, Google started to crawl pages using HTTPS as soon as we switched the scheme. No more pages were crawled using the HTTP scheme.

*Insert Mouad images here*

However, that doesn't mean that Google had replaced all the URLs in its index by the HTTPS version. It simply means that it continued to crawl using HTTP and was redirected to HTTPS.  
We noticed the HTTPS version of the main pages appeared in Google results **one day after the switch**.  
Then, around **80% of the indexed pages** had been replaced after 1 month.  
The deeper pages had been replaced in the Google index after about 3 months. 

### Traffic disturbances

Three months after the switch, our traffic from Google recovered to its normal trend.  
E-commerce sites migrating from HTTP to HTTPS can lose 40% of their traffic during several months. A little Market and A little Mercerie lost much less traffic and only for three months.  


**Etsy France's technical team successfully performed that big challenge thanks to several measures including SEO.**

Thanks for reading!
