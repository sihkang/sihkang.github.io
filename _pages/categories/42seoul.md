---
layout: archive
permalink: 42seoul
title: "42 Seoul"

author_profile: true
sidebar:
  nav: "docs"
---

{% assign posts = site.categories.42seoul %}
{% for post in posts %}
  {% include custom-archive-single.html type=entries_layout %}
{% endfor %}