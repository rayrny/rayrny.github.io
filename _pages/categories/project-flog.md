---
layout: archive
title: 실시간 감정분석 화상회의 'FLOG'
permalink: /categories/project-flog
author_profile: true
sidebar:
  title: Posts
  nav: "sidebar-contents"
---

<h2> - {{ site.categories.FLOG | size }} 개의 포스트 </h2>
-----
{% assign posts = site.categories.FLOG %}


{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
