---
layout: page
title: CTF write-ups
---

<p class="message">
  CTF write-ups.
</p>

<div class="posts">
  {% for post in site.categories.ctf %}
  <div class="post">
    <h1 class="post-title">
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h1>

    <span class="post-date">{{ post.author}}</span>

    {{ post.author }}
    {% for tag in post.tags %}
    	Tag: {{tag}}
    {% endfor %}
  </div>
  {% endfor %}
</div>