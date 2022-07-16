---
layout: page
title: Tech Blog
description: 개발과 관련된 글만 간추렸습니다.
permalink: /tech/
---

<div class="tags-wrapper">
  <div class="clouds">
  {% for category in site.categories %}
    {%- unless site.data.tech.categories contains category[0] -%}
      {% continue %}
    {%- endunless -%}
    <a href="#{{ category[0] }}">#{{ category[0] }}</a>
  {% endfor %}
  </div>

  {% for category in site.categories %}
    {%- unless site.data.tech.categories contains category[0] -%}
      {% continue %}
    {%- endunless -%}
  <div class="item" id="{{ category[0] }}">
    <h3>#{{ category[0] }}</h3>
    {% for post in category[1] %}
      <a class="post" href="{{ post.url }}">
        <span class="title">{{ post.title }}</span>
        <span class="time">
          <time datetime="{{ post.date | date_to_xmlschema }}">
            {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
            {{ post.date | date: date_format }}
          </time>
        </span>
      </a>
    {% endfor %}
  </div>
  {% endfor %}
</div>
