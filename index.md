---
layout: default
---
<ul>
  {% for post in site.posts %}
   {% if post.category == 'posts' %}
    <li>      
        <a href="{{ post.url }}">{{ post.title | append: "/"}}</a> 
        {{ post.excerpt }}
    </li>
    {% endif %}
  {% endfor %}
</ul>
<br>

#### Quickly Click~~

{% for post in site.posts %}
  {% if post.istop == 'true' %}
<a href="{{ post.url }}" >{{ post.title | prepend: "/" | prepend: post.category }}</a>
  {% endif %}
{% endfor %}


