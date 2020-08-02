---
layout: page
title: "Blog"
permalink: /blog/
---

Mostly posts about offensive website security

### Categories

<div>
	<ul>
		<li>
	{% for category in site.categories %}
	  {% assign category_name = category[0] %}
	    <a href="{{ site.baseurl }}/blog/category/{{ category_name | slugify }}/">{{ category_name | replace: "-", " " }}</a> /
	{% endfor %}
		</li>
	</ul>
</div>

<!-- #### Tags

<div>
	<ul>
	{% for tag in site.tags %}
	  {% assign tag_name = tag[0] %}
	  <li>
	    <a href="/tag/{{ tag_name | slugify }}/">{{ tag_name | replace: "-", " " }}</a>
	  </li>
	{% endfor %}
	</ul>
</div> -->

## All posts

{% for post in site.posts %}
<article>
  <h2>
    <a href="{{ post.url }}">
      {{ post.title }}
    </a>
  </h2>
  <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time>
  {{ post.content | strip_html | truncatewords: 60 }} }}
</article>
{% endfor %}
