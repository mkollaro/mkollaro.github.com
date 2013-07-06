---
layout: default
title: May Bayes Be With You
---
{% include JB/setup %}

<h2>Posts</h2>
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

<h2>Projects</h2>
* [DestroyStack](https://github.com/mkollaro/destroystack) - Fail injection
   tests for OpenStack

