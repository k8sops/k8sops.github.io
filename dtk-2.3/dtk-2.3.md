---
layout: page
title: Kubernetes Basics
---
<ul>
{% for page in site.pages %}
    {% if page.category == "dtk-2.3" %}
        <li>Chap # {{ page.chapter }} - <a href="{{ page.url }}">{{ page.title }}</a></li><br/>
    {% endif %}
{% endfor %}
</ul>