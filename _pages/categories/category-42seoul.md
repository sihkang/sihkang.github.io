---
title: "42seoul"
layout: archive
permalink: categories/42seoul
author_profile: true
sidebar:
  nav: "docs"
# sidebar_main: true
---

{% assign posts = site.categories.42seoul %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
