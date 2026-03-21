---
layout: page
title: Projects
permalink: /projects/
---

<div class="content-index">
  {% assign projects = site.projects | sort: "title" %}
  {% for project in projects %}
    <article class="content-card">
      <p class="content-meta">
        {% if project.status %}
          <span>{{ project.status }}</span>
        {% endif %}
        {% if project.tags and project.tags.size > 0 %}
          <span class="content-sep">·</span>
          <span>{{ project.tags | join: ", " }}</span>
        {% endif %}
      </p>
      <h2><a href="{{ project.url | relative_url }}">{{ project.title }}</a></h2>
      <p>{{ project.content | markdownify | strip_html | normalize_whitespace | truncate: 190 }}</p>
    </article>
  {% endfor %}
</div>
