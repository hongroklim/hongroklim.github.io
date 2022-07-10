---
layout: page
title: Projects
description: 배움과 성장을 경험했던 순간들입니다.
permalink: /projects/
---

WIP

{% for project in site.projects %}
  <h2>
    <a href="{{ project.url }}">
      {{ project.title }}
    </a>
  </h2>
{% endfor %}
