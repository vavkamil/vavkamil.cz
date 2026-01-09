---
layout: page
title: "Blog"
permalink: /blog/
---

<style>
    a {
        text-decoration: none;
    }

    .talks-toc ul {
        list-style: none;
        padding-left: 0;
    }

    .talks-toc li {
        display: flex;
        align-items: baseline;
        gap: 0.75rem;
        margin: 0.25rem 0;
    }

    /* Fixed-width year column */
    .talks-toc-year {
        width: 3.5rem;
        color: #7CFF00;
        /* optional, match your theme */
        flex-shrink: 0;
    }

    /* Push event name to the right */
    .talks-toc-event {
        margin-left: auto;
        opacity: 0.8;
        white-space: nowrap;
    }
li {
    border-bottom: 1px dotted rgba(124, 255, 0, 0.2);
}
</style>
Mostly posts about web application security, ethical hacking, and security research:

<nav class="talks-toc" aria-label="Talks table of contents">
    <ul>
        {% for post in site.posts %}
            {% assign anchor = post.title | slugify %}
            {% assign year = post.date | date: "%Y" %}
            <li>
                <span class="talks-toc-year">{{ year }}</span>
                <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
                <span class="talks-toc-event"><small><small>
{% capture words %}
  {{ post.content | number_of_words | minus: 180 }}
{% endcapture %}

{% unless words contains "-" %}
  {% assign minutes = words | plus: 180 | divided_by: 180 %}

  {% if minutes < 10 %}
    {{ minutes }} minutes
  {% else %}
    {{ minutes }} minutes
  {% endif %}
{% endunless %}

                </small></small></span>

            </li>
        {% endfor %}
    </ul>
</nav>

<hr>
<br>

### Categories

<div><small>
	<ul>
		<li>
	{% for category in site.categories %}
	  {% assign category_name = category[0] %}
	    <a href="{{ site.baseurl }}/blog/category/{{ category_name | slugify }}/">{{ category_name | replace: "-", " " }}</a> /
	{% endfor %}
		</li>
	</ul></small>
</div>

<hr>
<br>

{% for post in site.posts %}
<article>
  {% assign slug = post.url
  | remove_first: "/blog/"
  | remove: "/" %}

  <h2>
    <a href="{{ post.url }}">
      {{ post.title }}
    </a>
  </h2>
  <a href="{{ post.url }}">
      <img src="{{ '/assets/img/posts/' | append: slug | append: '.png' | relative_url }}"
           align="right"
           height="230"
           alt="{{ post.title }}"
           title=""
           style="border: 1px solid rgba(124, 255, 0, 0.2);
                  margin: 0px 0px 40px 40px">
  </a>
  <h3><em>{{ post.categories | join: " - " }}</em></h3>
  <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date: "%B %d, %Y" }}</time>
  <span> | </span>
{% capture words %}
  {{ post.content | number_of_words | minus: 180 }}
{% endcapture %}

{% unless words contains "-" %}
  {{ words | plus: 180 | divided_by: 180 | append: " minutes to read" }}
{% endunless %}

  <br>
  <br><br>
  <em>{{ post.excerpt | strip_html | truncatewords: 100 }}</em>
</article>
<br><br><br>
<hr>
{% endfor %}
