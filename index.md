---
layout: default
---

# vavkamil@localhost:~$

Hi 👋,

welcome on my personal website! 
        "This is it... this is where I belong..."
        I know everyone here... even if I've never met them, never talked to
them, may never hear from them again... I know you all...

## Latest post

{% for post in site.posts limit:1 %}
<article>
  <h2>
    <a href="{{ post.url }}">
      {{ post.title }}
    </a>
  </h2>
  <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time>
  <p>{{ post.content | strip_html | truncatewords: 50 }}</p>
</article>
{% endfor %}

## Recent posts

{% for post in site.posts offset:1 limit:3 %}
<article>
  <h2>
    <a href="{{ post.url }}">
      {{ post.title }}
    </a>
  </h2>
  <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time>
  {{ post.content | strip_html | truncatewords: 10 }}
</article>
{% endfor %}