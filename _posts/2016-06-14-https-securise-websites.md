---
layout: post
title:  "How Etsy France switched all its sites to HTTPS"
date:   2016-07-15
categories: projects
tags: [security, https, http, migration, project]
author: antoine
---

When Etsy acquired A little Market, the migration of our websites to https which was in the box since few years became one of our priority project in mid 2015.  
But before starting such a big and impacting project, we went through a long preparation.

## Project preparation

### Why ?
The goal of this project was to force all our users to access our services through HTTPS connections. Google and other search engines had to only crawl our websites in HTTPS only to avoid duplicate contents.

### How
We want to switch smoothly and easily from HTTP to HTTPS. At the most we want to update few configuration values to do it progressively.
All the blocking points to use HTTPS will have to be fixed before the deadline.

### Acceptance criteria
All our applications must be accessible through https.  
All users accessing the websites through http must be redirected to https.  

### When to switch ?
A switch from HTTP to HTTPS can have a big impact on the search engines positionning. As we do not use paid marketing, a bad indexing in search engines and especially Google can have really bad consequences on our traffic. Thanks to previous experiences, our SEO consultant estimated that SEO traffic returns back to normal around 3 months after switching.  
Because we had several marketing operations during the year (winter and summer sales, private sales, Christmas, etc.) and we didn't want a big impact on our SEO traffic during these operations, we carefully listed the period when the switch could be effective. We only found two:

- February/March (traffic should return back to normal around May)
- July/August (traffic should return back to normal around October)


## Milestones

1. Dependencies
    1. List all the dependencies of our main applications (ex: assets, blogs, images, etc)
    2. These dependencies will have to be migrated first.

2. We know some our hostnames contain more than one subdomain level (ex: static.commun.alittlemarket.com) but we don't want to buy expensive certificates that cover every levels.
    1. List all the concerned hostnames
    2. Fix incompatible subdomains (ex: static.commun.alittlemarket.com -> static-commun.alittlemarket.com). Fix them means:
        1. Create new subdomains in apache
        2. Add them in [Route53](https://aws.amazon.com/route53/)
        3. Update our stack to use them instead of the previous ones

3. Monitoring
   Monitor HTTP and HTTPS traffic using Statsd

4. Hardcoded URLs  
   Many links to our applications within our codebase are hardcoded
    1. Determine each type of hardcoded URLs (ex: http://www.alittlemarket.com, http://CONSTANT, http://assets.alittlemarket.com, etc)
    2. Replace all the hardcoded URLs by the equivalent constant
        1. Project by project
        2. file type by file type (php, smarty, twig, etc.)
        3. URL type by URL type
        4. URLs in database

5. Google
    1. Google Search Console  
       Create new entries in this tool
    2. Google Analytics  
       Try to differenciate HTTP traffic to HTTPS traffic

6. Force HTTPS
   Requests going through HTTP must be redirected to HTTPS  
   We don't want to block HTTP traffic

## Execution

- *How many time?*
- *Difficulties ?*
- *Unplanned tasks ?*

## Gradual activation

One of our goal was to progressively enable https on our applications, we didn't want to enable it for everybody at once.  
But on our websites for obscur reasons some of the oldest pages use the html tag `<base href="http://www.alittlemarket.com/" />` that we didn't succeeded in removing without breaking many links everywhere. The link in this tag is managed by configuration and couldn't change in function of the user.

We decided to hack this feature to be able to adapt this link in function of the consulted URL. In other words, if a user consults the website in http the link has to be http://www.alittlemarket.com and if a user consults the website in https the link has to be https://www.alittlemarket.com.
So, a user arriving in https on one of our websites stay in https all along his navigation.
It looked like:

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

The first step was to enable https on our Princess environment.  
Everybody who wants to deploy a feature in production (it arrives several times a day) must test it first on Princess. So, everybody was testing https on Princess without thinking about it during several days.

The Core team and our QA team had also tested https during all the process.

### Activation on production (for admin only)

After 1 week of tests on Princess, we decided to enable the feature on production but just for the admins.  
So, when someone was detected as being an admin and was surfing on http, he was automatically redirected to https by the application layer.

### Activation for everybody application by application

The big day was **February 29th**.  
The plan was to:

* Create a war room containg one person of each service (customer service, communication, technical teams, QA, SEO members, etc.)
* Switch the application one by one (alittlemarket.it, alittlemarket.com, alittlemercerie.com)
* After each switch, everybody in the war room had to check that everything goes right (metrics, functional tests, social networks, forums, etc.)
* Drink some beers after that big day !

#### Metrics

As you can see on the metrics below, we monitor global http and https traffic, the traffic for each website and Google bot traffic.

On the 3 graphs ALT IT, ALM FR and ALME FR we saw that the http traffic abruptly stops and at the same moment the https traffic starts. That's when we switched.  
On the global traffic we saw the same impact at the same times.  

And finally we saw a strange thing on the Google bot traffic. At 2pm (14:00) we see that Google bot http traffic stops but the https traffic does not start. It's not normal and we were worried about that. We couldn't afford to loose SEO. After some researches we discovered that Google had cached the robots.txt file we served it when it came through https (it was a different file than the http robots.txt file). In that file to avoid duplicated content we disallowed everything, that's why it stopped to crawled them.  
Forunately we discored that problem very quickly and only one hour and a half Google crawled our applications again. We only send again the good robots.txt in [Google Search Console](https://www.google.com/webmasters/tools/).

{:.text-center}
![Stasd metrics](/assets/https-securise-websites/grafana-traffic.png)

## What happened after ?



Thanks for reading!

