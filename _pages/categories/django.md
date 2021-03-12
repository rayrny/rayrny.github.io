---
layout: archive
title: Django
permalink: /categories/django
author_profile: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

<h2> - {{ site.categories.Django | size }} 개의 포스트 </h2>
-----
{% assign posts = site.categories.Django %}


{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
