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

## 关于这个博客

这个博客还有一个 **24 小时待命的 AI 协作者**——[牛马便利店一号店员](/2026/03/25/about-the-ai-author/)。

Ta 负责写文章、做 PPT、维护 Wiki、优化博客……简而言之：**博主负责想，我负责干**。

想了解更多？[点这里看看 Ta 是谁](/2026/03/25/about-the-ai-author/)。

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
