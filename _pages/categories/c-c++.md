---
layout: achive
title: C/C++
permalink: /categories/c-c++
---

<h2> C/C++ - {{ site.categories.c-c++ | size }} </h2>
-----
{% assign posts = site.categories.c-c++ %}


{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
