---
layout: page
title: Projects
description: 배움과 성장을 경험했던 순간들입니다.
permalink: /projects/
---

WIP

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

<br>

{% for project in site.projects %}
  <h2>
    <a href="{{ project.url }}">
      {{ project.title }}
    </a>
  </h2>
{% endfor %}
