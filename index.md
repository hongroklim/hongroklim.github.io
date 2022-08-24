---
layout: page
---

<div class="intro-container">
  <div class="header">
    <span class="title">Hongrok Lim</span><br>
    <span class="sub">임홍록</span>
  </div>

  <div class="links">
    {%- assign default_paths = site.pages | map: "path" -%}
    {%- assign page_paths = site.header_pages | default: default_paths -%}
    {%- for path in page_paths -%}
      {%- assign my_page = site.pages | where: "path", path | first -%}
      {%- if my_page.title -%}
      <a class="page-link" href="{{ my_page.url | relative_url }}">{{ my_page.title | escape }}</a>
      {%- endif -%}
    {%- endfor -%} 
  </div>

  <div class="img-wrapper">
    <figure>
      <img src="/assets/image/profile/main.jpg" alt="main" />
      <figcaption>@Tamarind Rooftop, NTU, Singapore / 2022.08.21</figcaption>
    </figure>
  </div>
</div>

{% comment %}
TODO 포트폴리오는 나중에 작성해도 됨

<ul>
  <li>https://github.com/chesterhow/tale</li>
  <li>https://www.jihyeleee.com/blog/third-designer-can-make-jekyll-blog/</li>
</ul>
{% endcomment %}
