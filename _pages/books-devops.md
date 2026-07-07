---
permalink: /books/devops/
title: "DevOps 读书笔记"
excerpt: "CI/CD 与 GitOps 实践"
author_profile: true
layout: default
---

{% assign category = nil %}
{% for cat in site.data.books.categories %}
  {% if cat.slug == "devops" %}
    {% assign category = cat %}
  {% endif %}
{% endfor %}

# {{ category.icon }} {{ category.title }}

**{{ category.subtitle }}**

{{ category.description }}

[← 返回读书笔记总览](/books/)

---

## 📋 书目列表

{% for entry in category.entries %}
<div class="books-entry-card {% if entry.status == 'completed' %}books-entry-completed{% elsif entry.status == 'reading' %}books-entry-reading{% else %}books-entry-planned{% endif %}">
  <div class="entry-status-badge {% if entry.status == 'completed' %}status-completed{% elsif entry.status == 'reading' %}status-reading{% else %}status-planned{% endif %}">
    {% if entry.status == "completed" %}已读{% elsif entry.status == "reading" %}在读{% else %}待读{% endif %}
  </div>
  <div class="entry-content">
    <h4>{% if entry.link and entry.link != "" %}[{{ entry.title }}]({{ entry.link }}){% else %}{{ entry.title }}{% endif %}</h4>
    <p class="entry-date">📅 {{ entry.date | date: "%Y-%m-%d" }}</p>
    <p class="entry-excerpt">{{ entry.excerpt }}</p>
    <div class="entry-tags">
      {% for tag in entry.tags %}
      <span class="entry-tag">{{ tag }}</span>
      {% endfor %}
    </div>
  </div>
</div>
{% endfor %}
