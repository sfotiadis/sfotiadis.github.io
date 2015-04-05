---
layout: page
title: Welcome
---
{% include JB/setup %}

## About

I'm Stathis, welcome to my personal website.

## Recent blog posts

<ul class="posts">
  {% for post in site.posts limit:2 %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


