---
layout: default
---
<ul>
  {% for post in site.posts %}
   {% if post.category == 'posts' %}
    <li>      
        <a href="{{ post.url }}">{{ post.title }}</a> 
        {{ post.excerpt }}
    </li>
    {% endif %}
  {% endfor %}
</ul>





