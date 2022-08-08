---
layout: page
title: Projects
description: 배움과 성장을 경험했던 순간들입니다.
permalink: /projects/
---

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

<div class="projects-wrapper">
  <ul>
    {% assign projects = site.projects | where_exp:"e", "e.order >= 0" | sort: "order" %}
    {% for project in projects %}
      <li class="item">
        <a href="{{ project.url }}">
          <span class="spec">{{ project.spec }}</span>
          <span class="title">{{ project.title }}</span>
          <span class="excerpt">{{ project.excerpt }}</span>
        </a>
      </li>
    {% endfor %}
  </ul>
</div>
