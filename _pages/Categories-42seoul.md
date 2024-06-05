---
permalink: /42seoul
title: "42 Seoul"
toc: true
toc_sticky: true
toc_label: "42 Seoul"
layout: archive
---

{% assign posts = site.categories.42seoul %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}