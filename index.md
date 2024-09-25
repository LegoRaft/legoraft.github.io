---
layout: default
title: Home
---

<h1 class="posttitle">Posts</h1>

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a class="postlink" href="{{ post.url }}">{{ post.title }}</a></h2>
    </li>
  {% endfor %}
</ul>