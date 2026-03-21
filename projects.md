---
layout: page
title: Projects
permalink: /projects/
---

{% for project in site.projects %}
- [{{ project.title }}]({{ project.url }})
  {%- if project.status %} — <small>{{ project.status }}</small>{% endif %}
{% endfor %}
