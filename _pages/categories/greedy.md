---
layout: archive
title: Greedy
permalink: /categories/greedy
---

<h2> Greedy - {{ site.categories.greedy | size }} </h2>
-----
{% assign posts = site.categories.greedy %}


{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
