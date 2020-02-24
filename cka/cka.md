---
layout: page
title: CKA Notes
---
{% assign sorted_pages = site.pages | sort:"chapter" %}

<ul>
{% for page in sorted_pages %}
    {% if page.category == "cka" %}
        <li>Chap # {{ page.chapter }} - <a href="{{ page.url }}">{{ page.title }}</a></li><br/>
    {% endif %}
{% endfor %}
</ul>