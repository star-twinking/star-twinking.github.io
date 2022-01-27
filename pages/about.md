---
layout: page
title: About
description: Recoding What I Learn.
keywords: recording, learning
comments: true
menu: 关于
permalink: /about/ 
---

Recording life.

## 联系

dql0617@foxmail.com



## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
