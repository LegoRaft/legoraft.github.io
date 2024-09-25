---
layout: default
title: Home
---

<h1 class="posttitle">Posts</h1>

<ul class="postlist">
  {% for post in site.posts %}
    <li>
      <span class="date">{{ post.date | date_to_string }}</span>
      <a class="postlink" href="{{ post.url }}">
        <h3>{{ post.title }}</h3>
        <p>{{ post.excerpt }}</p>
      </a>
    </li>
  {% endfor %}
</ul>