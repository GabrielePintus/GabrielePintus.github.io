---
layout: page
title: Posts
permalink: /posts/
---

<div class="content-index">
  {% for post in site.posts %}
    <article class="content-card">
      <p class="content-meta">
        <span>{{ post.date | date: "%B %d, %Y" }}</span>
        {% if post.categories and post.categories.size > 0 %}
          <span class="content-sep">·</span>
          <span>{{ post.categories | join: ", " }}</span>
        {% endif %}
      </p>
      <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
      <p>{{ post.excerpt | strip_html | normalize_whitespace | truncate: 190 }}</p>
    </article>
  {% endfor %}
</div>
