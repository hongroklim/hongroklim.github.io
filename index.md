---
layout: page
---

<div class="intro-container">
  <div class="header">
    <span class="title">Hongrok Lim</span>
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
      <figcaption>@Ubud, Bali, Indonesia / 2022.09.27</figcaption>
    </figure>
  </div>
</div>
