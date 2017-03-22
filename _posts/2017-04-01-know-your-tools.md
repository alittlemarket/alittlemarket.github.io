---
layout: post
title:  "Know your tools : the Composer case"
date:   2017-03-01
categories: tools
tags: ["composer", "dependency management", "dependency manager", "php"]
author: laurent
---

## Introducing dependency management

**Dependency Managers** are nowadays a masterpiece of every application lifecycles. Think about [Maven](https://maven.apache.org/) in Java ecosystem, [pip](https://pip.pypa.io/en/stable/) for Python, [npm](https://www.npmjs.com/) for Node.js and finally, [Composer](https://getcomposer.org/) for PHP.
Their main purposes are to install, upgrade and remove third-party libraries needed for applications to build and run. Those who don't use a **Dependency manager** surely never faced the [dependency hell](https://en.wikipedia.org/wiki/Dependency_hell) issue !

**Dependency Managers** are to _softwares_ where **Package Managers** are to _systems_ ; we can't no more do our job without them.

**Dependency management** concepts look simple and easy from the outside, because the complexity is hidden to keep usage as simple as possible. That's what about this post is : did engineer really know the tools they're using ?

## How EtsyFrance is using Composer

{:.text-center}
![Composer](/assets/dm/logo-composer-transparent2.png)

Here's a simplified view of our basecode structure and how [Composer](https://getcomposer.org/) make the glue :

```bash
#Webfronts git repository
- ALittleMarket
  \_ src/[...]
  \_ tests/[...]
  \_ web/[...]
  \_ composer.json
  \_ composer.lock
- ALittleMercerie
  \_ src/[...]
  \_ tests/[...]
  \_ web/[...]  
  \_ composer.json
  \_ composer.lock
- Common
  \_ src/[...]
  \_ tests/[...]
  \_ composer.json
```

### Root projects composer.json

In ALittleMarket/composer.json & ALittleMercerie/composer.json :

```json
{
    "name": "alm/ALittleMarket",
    "repositories": [
        {
            "type": "path",
            "url": "../Common"
        }
    ],
    "require": {
        "alm/common": "*@dev"
    }
}
```

`ALittleMarket` and `ALittleMercerie` are root projects. For our purpose, it means that's from where `composer install` in executed.

`alm/common` dependency is a [path](https://getcomposer.org/doc/05-repositories.md#path) repository : it means that we're using a local directory as a composer-aware package.

### Common/composer.json

`Common` package depends on `doctrine/orm` and doesn't have a committed `composer.lock` file, because it's not the `root` project :

```json
{
    "name": "alm/common",
    "require": {
        "doctrine/orm": "2.4.*"
    }
}
```

## Case : upgrade doctrine/orm

We need to upgrade `doctrine/orm` to `2.5.*` to benefit a feature that doesn't exist in `2.4` branch.

Before beginning the upgrade, let's see at which versions dependencies are locked :

```bash
$ composer show
alm/common                dev-master
doctrine/annotations      v1.4.0     Docblock Annotations Parser
doctrine/cache            v1.6.1     Caching library offering an object-oriented API for many cache backends
doctrine/collections      v1.4.0     Collections Abstraction library
doctrine/common           v2.7.2     Common Library for Doctrine projects
doctrine/dbal             v2.4.5     Database Abstraction Layer
doctrine/inflector        v1.1.0     Common String Manipulations with regard to casing and singular/plural rules.
doctrine/lexer            v1.0.1     Base library for a lexer that can be used / Top-Down, Recursive Descent Parsers.
doctrine/orm              v2.4.8     Object-Relational-Mapper for PHP
psr/log                   1.0.2      Common interface for logging libraries
symfony/console           v2.8.18    Symfony Console Component
symfony/debug             v3.0.9     Symfony Debug Component
symfony/polyfill-mbstring v1.3.0     Symfony polyfill for the Mbstring extension
```

As we need choose to upgrade `doctrine/orm` to a very specific version - `2.5.6` -, let's update `Common/composer.json` :

In Common/composer.json :

```json
{
    "name": "alm/common",
    "require": {
        "doctrine/orm": "2.5.6"
    }
}
```

### Naive update approach

```bash
$ cd ALittleMarket
$ composer update doctrine/orm
```
![nothing-to-install-or-update](/assets/dm/composer-nothing-to-install-or-update.png)

Nothing to update. Why ?

> Composer know nothing about `doctrine/orm`. The only dependency it's aware of is its [transitive dependencies](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Transitive_Dependencies) : `alm/common`.

At this point, another engineer stops behind me :

> Need help with Composer ?

```bash
$ cd ALittleMarket
$ composer update alm/common
```
![nothing-to-install-or-update](/assets/dm/composer-nothing-to-install-or-update.png)

Nothing to update. Why ?

> From Composer perspective, `alm/common` - as it's a path repository unaware of its `version` - can only be installed, it doesn't make sense to ask for an update.

At this point, a third engineer stops behind me :

> Did you try with --with-dependencies ?

```bash
$ cd ALittleMarket
$ composer update alm/common --with-dependencies
```
![too-many-things-to-update](/assets/dm/composer-update-wd.png)

This time, there's too many dependencies updated. Also, `symfony/console (v2.8.18 => v3.2.6)` updated itself to a new major version. That's not what I wanted.

> --with-dependencies flatten the transitive dependencies of the whitelisted packages and try to update them **all**

At this point, everything we're trying are just _guesses_ & _blind tries_ :

```bash
$ cd ALittleMarket
$ composer update alm/common doctrine/orm
```

```bash
Problem 1
    - Conclusion: don't install doctrine/orm v2.5.6|remove doctrine/dbal v2.4.5|install doctrine/dbal v2.5.0|install doctrine/dbal v2.5.1|install doctrine/dbal v2.5.2|install doctrine/dbal v2.5.3|install doctrine/dbal v2.5.4|install doctrine/dbal v2.5.5
    - Conclusion: don't install doctrine/orm v2.5.6|don't install doctrine/dbal v2.4.5|install doctrine/dbal v2.5.0|install doctrine/dbal v2.5.1|install doctrine/dbal v2.5.2|install doctrine/dbal v2.5.3|install doctrine/dbal v2.5.4|install doctrine/dbal v2.5.5
    - Installation request for doctrine/common (locked at v2.7.2) -> satisfiable by doctrine/common[v2.7.2].
    - Conclusion: don\'t install doctrine/orm v2.5.6|remove doctrine/dbal v2.4.5|install doctrine/dbal v2.5.0|install doctrine/dbal v2.5.1|install doctrine/dbal v2.5.2|install doctrine/dbal v2.5.3|install doctrine/dbal v2.5.4|install doctrine/dbal v2.5.5
    - Conclusion: don\'t install doctrine/orm v2.5.6|don't install doctrine/dbal v2.4.5|install doctrine/dbal v2.5.0|install doctrine/dbal v2.5.1|install doctrine/dbal v2.5.2|install doctrine/dbal v2.5.3|install doctrine/dbal v2.5.4|install doctrine/dbal v2.5.5
    - Installation request for doctrine/dbal (locked at v2.4.5) -> satisfiable by doctrine/dbal[v2.4.5].
    - alm/common dev-master requires doctrine/orm 2.5.6 -> satisfiable by doctrine/orm[v2.5.6].
    - alm/common dev-master requires doctrine/orm 2.5.6 -> satisfiable by doctrine/orm[v2.5.6].
    - alm/common dev-master requires doctrine/orm 2.5.6 -> satisfiable by doctrine/orm[v2.5.6].
    - Conclusion: don't install doctrine/orm v2.5.6
    - Installation request for alm/common *@dev -> satisfiable by alm/common[dev-master].
```

Update failed again. Conclusions drawn and displayed by **Composer** are not really helping me, because they don't explain what to do. At least, I know there's an issue with `doctrine/dbal`. I guess that `doctrine/orm 2.5.6` needs `doctrine/dbal` to be updated.

### What the hell with ... ?

Composer ? Or with three engineers stuck on a _detail_ ?

Some "What the hell about Composer ?" later, it suddenly blow in my mind :

> A poor workman always blames his tools

Shame. I realized that, since I use *Composer* - from the early days, I must say - I never took time to read the whole manual, just very targeted parts, like we all did with a dictionnary for instance.

It's time to RTM, isn't it ? That's where I found the **Whitelist** concept, which is not very well explained in the documentation, but make the whole picture very clear :

```bash
$ cd ALittleMarket
$ composer update alm/common doctrine/orm doctrine/dbal
```
![bingo](/assets/dm/composer-bingo.png)

This time, everything is updated as expected :

- Installing `doctrine/instantiator`, a new dependency of the new `doctrine/dbal` package.
- Updating `doctrine/dbal` (v2.4.5 => v2.5.12), upgraded to the highest possible version, according to dependency graph.
- Updating `doctrine/orm` (v2.4.8 => v2.5.6), upgraded to the exact version we wanted.
- Removing `alm/common` (dev-master)
- Installing `alm/common` (dev-master) Symlinked from ../Common

### What did we learn ?

We learnt that :

* We used a critical tool without real deep knowledge of how it works. We were surprised about that fact.
* For each tool we use on a daily basis, we need to find at least one person in our team who knows the tool or is ready to get deeper understanding
* Read The Manuals !

And you, as a daily **Composer** user, did you read the whole documentation ?
