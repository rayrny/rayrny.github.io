---
layout: achive
title: Git Blog
permalink: /categories/blog
---

<h2> Ect. - {{ site.categories.ect | size }} </h2>
-----
{% assign posts = site.categories.ect %}


{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
