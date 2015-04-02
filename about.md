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

{% if forloop.index == 2 %}
{:.d-iblock.padded-all.text-center.about-team-member}
<img class="d-block spaced-auto spaced-bottom" src="{{ member.avatar }}" width="50" height="50" alt="{{ member.github }}" />
[{{ member.name }}](https://github.com/{{ member.github }})
{% endif %}


{% endfor %}
{% endfor %}
