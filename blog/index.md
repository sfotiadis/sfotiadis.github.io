---
layout: page
title: My blog
---
{% include JB/setup %}
    
### Blog

<br>
This blog has been around since 2012. It's where I occasionally write about things of my interest. 
Initially it was hosted on blogger but from April 2015 it has been migrated to Jekyll. All the old posts have been transferred.

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


