---
layout: post
title:  "How we saved time with Mosaico"
date:   2016-08-26
categories: projects
tags: [mosaico, tooling, emailing, newsletter]
author: celine
---

Whenever product owners want to implement a new feature or an improvement, they need to ask for engineering resources.
No matter how great or inovative their idea is, they just depend on the tech team and how the latter prioritises the project.


That is hugely frustrating for everybody because, of course, there are several projects going on and each team have their own goals.
It's more and more difficult to juggle between projects and sometimes you just have to answer "not until next month" or "not this year" or even sometimes,
just "that's not possible". This is a curse for most of startups and small companies.

## Invest on tooling

### Two goals

Whenever it's possible, one solution to help unlock that kind of situation is to invest some time into developing a tool
that will give non-tech workmates more autonomy.

This is what we decided to do for our emails. PR & communication, E-merchandising, Apps, Events : a handfull of different
teams need to send occasional or regular e-mails to our users. A little Market emailing initial codebase is old and most of **the emails
are not responsive, they needed to be rebuilt**. But there must be dozens of emails and this seems like a big Technical debt project, right?

On top of that, **their design needed freshening up** as it wasn't consistent to our branding. It did not reflect our values either,
so we wanted it merrier and more humane.

But there can't be a dedicated engineer to refactor every email, or to build/correct a new one each week.

Instead of coding a new email everytime we needed to send one, we gathered all the needs and constraints expressed by
the teams that use emails the most. The idea was to be able to build a *WYSIWYG* email templating interface that would provide all kinds of
content blocks and give anyone the possibility to use those modules without having to open an IDE nor dig in the HTML.
They'd drag-and-drop them, move them, duplicate them, delete them at will, and then they'd file in the desired content
(texts, images, links). Once it's done, they'd download the HTML result and go back to their usual workflow
(test it and when everything's fine, schedule it.)


Mosaico is a tool that provides modules or blocks that can be created, removed, duplicated and personalized easily, regarding each team's needs.

### How [Mosaico](https://mosaico.io/) works


{:.text-center}
![Mosaico](/assets/tooling-mosaico/mosaico-logo.png)

Behind this full Table+CSS logo, there's a project lead by [Voidlabs](https://github.com/voidlabs/mosaico). It's built with knockoutJS and it provides
three examples of fully functioning master templates with their own design and blocks.
Through knockout syntax and a special use of the @support query in the head <style> tag, it allows to bind the blocks
and their content to multiple properties: direct text edition, theme and size customization, display options, image placeholders, etc.

Building our own master template was but the best occasion to start from scratch and make sure that those emails would have
a brand new design *and also* a seriously tested responsive HTML structure.