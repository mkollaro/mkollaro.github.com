---
layout: default
title: Random tech notes
---
{% include JB/setup %}

<h2>Posts</h2>
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

<h2>Projects</h2>
* [NSim](https://github.com/mkollaro/nsim) - physics simulator of the Solar
  system (or any other) with both CSV and OpenGL output
* [DestroyStack](https://github.com/mkollaro/destroystack) - Fault injection
   tests for OpenStack
* [LaunchpadStats](https://github.com/mkollaro/launchpadstats) - create tables out
  of launchpad statistics (e.g how many commits each user has)
* [Taskrunner](https://github.com/mkollaro/taskrunner) - very simple executor of
  Python scripts, made to replace a more complicated tool
* [C snippets](https://github.com/mkollaro/c_snippets) - experiments and
  exercises in C
* [OpenGL snippets](https://github.com/mkollaro/c_snippets) - experiments and
  reproducers for OpenGL bugs


<h2>Contact</h2>
* irc: mkollaro on freenode.net
* email: mkollaro (AT) gmail (DOT) com

