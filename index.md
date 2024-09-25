---
layout: default
title: Home
---

# Posts

<ul>
  {% for post in site.posts %}
    <li>
      <h2 class="postlink"><a href="{{ post.url }}">{{ post.title }}</a></h2>
    </li>
  {% endfor %}
</ul>