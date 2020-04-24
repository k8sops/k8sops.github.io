---
layout: page
title: Open Shift
---
{% assign sorted_pages = site.pages | sort:"chapter" %}

<ul>
{% for page in sorted_pages %}
    {% if page.category == "openshift" %}
        <li>Chap # {{ page.chapter }} - <a href="{{ page.url }}">{{ page.title }}</a></li><br/>
    {% endif %}
{% endfor %}
</ul>