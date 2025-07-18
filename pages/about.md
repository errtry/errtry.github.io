---
layout: page
title: About
description: 牛马假日
keywords: 牛马, 牛马假日
comments: true
menu: 关于
permalink: /about/
---

记录一些可以记录的事物，避免遗忘。毕竟牛马我通常容易遗忘....TvT

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
</ul>


## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
