---
layout: page
title: Kubernetes Basics
---
<li>
{% for page in site.pages %}
    {% if page.category == "dtk-2.3" %}
        <a href="{{ page.url }}">{{ page.title }}</a>
    {% endif %}
{% endfor %}
</li>