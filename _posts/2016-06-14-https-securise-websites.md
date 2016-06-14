---
layout: post
title:  "Use https to securise our services"
date:   2016-06-14
categories: security
tags: [security, https, http, migration]
author: antoine
---

When Etsy acquired A little Market, the migration of our websites to https which was in the box since few years became one of our priority project in mid 2015.  
But before starting such a big and impacting project, we went though a long preparation.

## Project preparation

### Why ?
The goal of this project was to force all our users to access our services through HTTPS connections. Google and other search engines had to only crawl our websites in HTTPS only to avoid duplicate contents.

### When to switch ?
A switch from HTTP to HTTPS can have a big impact on the search engines positionning. As we do not use paid marketing, a bad indexing in search engines and especially Google can have really bad consequences on our traffic. Thanks to previous experiences, our SEO consultant estimated that SEO traffic returns back to normal around 3 months after switching.  
Because we had several marketing operations during the year (winter and summer sales, private sales, Christmas, etc) and we didn't want a big impact on our SEO traffic during these operations, we carefully listed the period when the switch could be effective. We only found two:

- February/March
- July/August

### Milestones

1. Dependencies
    1. List all the dependencies of our main applications (ex: assets, blogs, images, etc)
    2. These dependencies will have to be migrated first.

2. We know some our hostnames contain more than one subdomain level (ex: static.commun.alittlemarket.com) but we don't want to buy certificates that cover every levels.
    1. List all the concerned hostnames
    2. Fix incompatible subdomains (ex: static.commun.alittlemarket.com -> static-commun.alittlemarket.com). Fix them means:
        1. Create new subdomains in apache
        2. Add them in Route53
        3. Update our stack to use them instead of the previous ones

3. Hardcoded URLs  
   Many links to our applications within our codebase are hardcoded
    1. 



## Execution


## Gradual activation


## What happened after ?

