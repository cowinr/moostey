---
layout: default
title: Blog Posts
---

{% for post in site.posts %}| {{ post.date | date_to_string }} | <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a> |
{% endfor %}
