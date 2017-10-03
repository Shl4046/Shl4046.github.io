---
layout: default
---

Text can be **bold**, _italic_, or ~~strikethrough~~.

[back](./)
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>


