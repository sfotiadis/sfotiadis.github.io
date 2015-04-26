---
layout: page
title: Welcome
---
{% include JB/setup %}

![photo of Stathis in Alhambra](assets/images/stathis.jpg){: .inset .right style="width: 120px"}

# *Using silicon based machines to serve carbon based ones.*

I'm Stathis, welcome to my personal website. 

## About

I'm a physicist and computer scientist by education and a data scientist by profession. 

I like <a href="projects">coding</a> to discover things. Also from time to time I like to write down my thoughts in my <a href="/blog">blog</a>.

I'm also very interested in machine learning and neuroscience. What fascinates me is their intersection especially from an information theoretical and probabilistic inference point of view.

## Around the web

Check my CV on Linkedin
<script src="//platform.linkedin.com/in.js" type="text/javascript"></script>
<script type="IN/MemberProfile" data-id="https://www.linkedin.com/in/stathisfotiadis" data-format="inline" data-related="false"></script>

Follow me on twitter
<a class="twitter-timeline" href="https://twitter.com/sfotiadis_" height="2em" data-widget-id="586226304086847488" data-chrome="nofooter noborders noscrollbar transparent noheader" data-tweet-limit="1" data-show-replies="false">Tweets by @sfotiadis_</a>
<a class="twitter-follow-button"
  href="https://twitter.com/sfotiadis_"
  data-show-count="false"
  data-lang="en">
</a>
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+"://platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>


Read my answers and questions at Stack Overflow
<a href="http://stackoverflow.com/users/1857521/sfotiadis">
<img src="http://stackoverflow.com/users/flair/1857521.png?theme=clean" width="208" height="58" alt="profile for sfotiadis at Stack Overflow, Q&amp;A for professional and enthusiast programmers" title="profile for sfotiadis at Stack Overflow, Q&amp;A for professional and enthusiast programmers">
</a>

Fork my code on [Github](https://github.com/sfotiadis).<br>

## Recent blog posts

<ul class="posts">
  {% for post in site.posts limit:3 %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

For more please visit my <a href="/blog">blog</a>.


