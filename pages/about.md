---
layout: page
title: About
description: 打码改变世界
keywords: Zhuang Ma, 马壮
comments: true
menu: 关于
permalink: /about/
---


# Hey，我是Luck

- 毕业于南少林，擅长打野升级.
- 喜欢Linux，热爱开源。Game: 绝地求生，王者荣耀
- 梦想：做一个有钱和有品味的人，实在不行，只做个有钱人也可以
- 目的：分享工作中的一些经验以及学习到的一些技能，在下笔拙，但非常愿意和大家分享一些干货。有什么困难尽管跟我说，我会尽自己最大能力.

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}

## 联系

- 邮箱：bighug.top@gmail.com

- <img src="http://ocppiicaw.bkt.clouddn.com/me.jpg"  alt="扫我微信" />
