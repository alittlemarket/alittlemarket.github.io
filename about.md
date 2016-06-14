---
layout: page
title:  "Meet ALM Tech Team"
date:   2015-03-26 14:34:25
categories: People
tags: [ alittlemarket, team]
author: celine
permalink: /about/
---

Meet the ALM team !


{% for members in site.authors %}
{% for member in members %}

{% if forloop.index == 2 and member.active != false %}
{:.d-iblock.padded-bottom.spaced-top.text-center.about-team-member}
<img class="d-block spaced-auto spaced-bottom" src="{{ member.avatar }}" width="100" height="100" alt="{{ member.github }}" />
[{{ member.name }}](https://github.com/{{ member.github }})
{% endif %}


{% endfor %}
{% endfor %}
