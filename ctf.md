---
layout: page
title: CTF write-ups
---

<hr>
<div class="posts">
  {% for post in site.categories.ctf %}
  <div class="post">
    <h1 class="post-title">
      <a href="{{ post.url }}">
        {{post.event}} - {{ post.title }}
      </a>
    </h1>
    
    <span class="post-date">{{ post.date | date_to_string }}</span>
    <blockquote>
      {{ post.description }}
    </blockquote>
  </div>
  {% endfor %}
</div>
