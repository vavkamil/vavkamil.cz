---
layout: page
title: "Blog category: Public talks"
name: Public talks
---

xxx

<div>
	<ul>
	{% for post in site.posts %}
		{% if post.categories contains page.name %}
		<li>
		<h3>
		<a href="{{ post.url }}">
		{{ post.title }}
		<small>{{ post.date | date_to_string }}</small>
		</a>
		</h3>
		</li>
		{% endif %}
	{% endfor %}
	</ul>
</div>