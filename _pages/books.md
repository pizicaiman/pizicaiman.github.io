---
permalink: /books/
title: "读书笔记"
excerpt: "技术书籍阅读笔记与知识体系整理"
author_profile: true
layout: default
---

# 📚 读书笔记

技术书籍阅读笔记与知识体系整理，涵盖 AIOps、DevOps、SRE 等领域的核心著作与实践指南。

---

## 📊 阅读看板

{% assign total_entries = 0 %}
{% assign completed_count = 0 %}
{% assign reading_count = 0 %}
{% assign planned_count = 0 %}
{% for category in site.data.books.categories %}
  {% for entry in category.entries %}
    {% assign total_entries = total_entries | plus: 1 %}
    {% if entry.status == "completed" %}
      {% assign completed_count = completed_count | plus: 1 %}
    {% elsif entry.status == "reading" %}
      {% assign reading_count = reading_count | plus: 1 %}
    {% elsif entry.status == "planned" %}
      {% assign planned_count = planned_count | plus: 1 %}
    {% endif %}
  {% endfor %}
{% endfor %}

<div class="books-dashboard-stats">
  <div class="stat-card stat-total">
    <div class="stat-number">{{ total_entries }}</div>
    <div class="stat-label">总计书目</div>
  </div>
  <div class="stat-card stat-completed">
    <div class="stat-number">{{ completed_count }}</div>
    <div class="stat-label">已读完</div>
  </div>
  <div class="stat-card stat-reading">
    <div class="stat-number">{{ reading_count }}</div>
    <div class="stat-label">在读中</div>
  </div>
  <div class="stat-card stat-planned">
    <div class="stat-number">{{ planned_count }}</div>
    <div class="stat-label">待阅读</div>
  </div>
</div>

---

## 📂 分类总览

<div class="books-category-grid">
{% for category in site.data.books.categories %}
  {% assign cat_total = category.entries | size %}
  {% assign cat_completed = 0 %}
  {% for entry in category.entries %}
    {% if entry.status == "completed" %}
      {% assign cat_completed = cat_completed | plus: 1 %}
    {% endif %}
  {% endfor %}
  <a href="{{ category.permalink }}" class="books-category-card">
    <div class="category-card-icon">{{ category.icon }}</div>
    <div class="category-card-content">
      <h3>{{ category.title }}</h3>
      <p class="category-card-subtitle">{{ category.subtitle }}</p>
      <p class="category-card-desc">{{ category.description }}</p>
      <div class="category-card-stats">
        <span class="category-count">{{ cat_total }} 本书</span>
        <span class="category-progress">{{ cat_completed }}/{{ cat_total }} 已读</span>
      </div>
      <div class="category-progress-bar">
        {% if cat_total > 0 %}
        <div class="progress-fill" style="width: {{ cat_completed | times: 100 | divided_by: cat_total }}%"></div>
        {% endif %}
      </div>
    </div>
  </a>
{% endfor %}
</div>

---

## 📖 最近更新

{% for category in site.data.books.categories %}
  {% for entry in category.entries %}
    {% if entry.status == "reading" %}
<div class="books-entry-card books-entry-reading">
  <div class="entry-status-badge status-reading">在读</div>
  <div class="entry-content">
    <h4>{{ entry.title }}</h4>
    <p class="entry-category">{{ category.icon }} {{ category.title }}</p>
    <p class="entry-excerpt">{{ entry.excerpt }}</p>
    <div class="entry-tags">
      {% for tag in entry.tags %}
      <span class="entry-tag">{{ tag }}</span>
      {% endfor %}
    </div>
  </div>
</div>
    {% endif %}
  {% endfor %}
{% endfor %}
