---
layout: default
---

# Hello world!

Welcome on my personal website!

## Latest post

{% for post in site.posts limit:1 %}
<article>
  <h2>
    <a href="{{ post.url }}">
      {{ post.title }}
    </a>
  </h2>
  <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time>
  <p>{{ post.content | strip_html | truncatewords: 60 }}</p>
</article>
{% endfor %}

## Recent posts

{% for post in site.posts offset:1 limit:2 %}
<article>
  <h2>
    <a href="{{ post.url }}">
      {{ post.title }}
    </a>
  </h2>
  <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time>
  {{ post.content }}
</article>
{% endfor %}