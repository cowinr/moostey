---
layout: default
title: Blog Posts
---

| Date | Post |
|----|
{% for post in site.posts %}| {{ post.date | date_to_string }} | <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a> |{% endfor %}
