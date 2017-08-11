---
layout: page
title: Book Summaries
---

<hr>
<p class="message">
  Summaries of books I read for quickly looking up that one detail I forgot but remembered as really useful.
</p>

<div class="posts">
  {% for post in site.categories.books %}
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