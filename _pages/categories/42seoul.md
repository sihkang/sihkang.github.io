---
layout: archive
permalink: 42seoul
title: "42seoul"

author_profile: true
sidebar:
  nav: "docs"
---

{% assign posts = site.categories.categories %}
{% for post in posts %}
  {% include custom-archive-single.html type=entries_layout %}
{% endfor %}