---
layout: page
title: Projects
description: 배움과 성장을 경험했던 순간들입니다.
permalink: /projects/
---

WIP

{% comment %}
* AI/DL
  * Sentiment Analysis (Pandas, Tensorflow)
  * Recycling Assistant (SSD MobileNet, OpenCV)
* Web
  * Elearning Py (Django, jQuery)
  * Quick Attendance (React)
  * IDE for spring (Docker, Wildfly, Postgresql)
* Big Data
  * Observatory (Scala, Spark)
  * DBMS in C++ (B+ Tree, Buffer, Locking, CC)
  * xv6-public (OS)
{% endcomment %}

<ul>
  {% assign projects = site.projects | where_exp:"e", "e.order >= 0" | sort: "order" %}
  {% for project in projects %}
    <li>
    <a href="{{ project.url }}">
      {{ project.title }}
    </a>
    </li>
  {% endfor %}
</ul>
