---
layout: default
title:  "languages"
category: "posts"
excerpt_separator: ""
---
<ul>
  {% for post in site.posts %}
   {% if post.category == 'languages' %}
    <li>      
        <a href="{{ post.url }}">{{ post.title }}</a> 
        {{ post.excerpt }}
    </li>
    {% endif %}
  {% endfor %}
</ul>