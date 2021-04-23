---
layout: archive
title: Python
permalink: /categories/python
author_profile: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

<h2> - {{ site.categories.Python | size }} 개의 포스트 </h2>
-----
{% assign posts = site.categories.python %}


{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}