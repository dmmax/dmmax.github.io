---
permalink: /coding
layout: single
title: Заметки программиста
---
<ul>
  {% for coding in site.coding %}
    <li>
      <h2><a href="{{coding.url}}">{{ coding.name }}</a></h2>
      <p>{{ coding.content | markdownify }}</p>
    </li>
  {% endfor %}
</ul>