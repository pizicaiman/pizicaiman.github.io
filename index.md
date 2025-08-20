// filepath: d:\Person\Projects\pizicaiman.github.io\index.md
---
layout: default
title: 首页
---

# 欢迎来到我的技术博客

这里记录我在云原生、微服务、Web3等领域的技术探索。

## 最新文章

{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url }}) - {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}