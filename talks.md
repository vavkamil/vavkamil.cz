---
layout: page
title: "Public talks"
permalink: /talks/
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

    .talks-toc-year {
        width: 3.5rem;
        color: #7CFF00;
        flex-shrink: 0;
    }

    .talks-toc-event {
        margin-left: auto;
        opacity: 0.8;
        white-space: nowrap;
    }
    
    li {
        border-bottom: 1px dotted rgba(124, 255, 0, 0.2);
    }

  @media (max-width: 640px) {
    .talks-toc li {
      display: grid;
      grid-template-columns: 4.5rem 1fr;
      column-gap: 0.75rem;
      row-gap: 0.25rem;
      align-items: start;
      min-width: 0;
      padding-bottom: 0.4rem;
    }

    .talks-year {
      grid-column: 1;
      grid-row: 2;
    }

    .talks-toc li > small {
      grid-column: 2;
      grid-row: 2;
      justify-self: start;
    }

    .talks-right {
      grid-column: 2;
      grid-row: 2;
      justify-self: end;
      white-space: nowrap;
      max-width: 100%;
      overflow: hidden;
      text-overflow: ellipsis;
      min-width: 0;
    }

    .talks-toc li > a {
      grid-column: 1 / -1;
      grid-row: 1;
      min-width: 0;
      overflow-wrap: anywhere;
      display: block;
      padding: 0.15rem 0;
    }
</style>
15+ talks. 13+ years. Mostly offensive web security, bug bounty, and security research:
{% assign talks_sorted = site.talks | sort: "latest_event_date" | reverse %}
<nav class="talks-toc" aria-label="Talks table of contents">
    <ul>
        {% for talk in talks_sorted %}
            {% assign anchor = talk.title | slugify %}
            {% assign year = talk.latest_event_date | date: "%Y" %}
            <li>
                <span class="talks-toc-year">{{ year }}</span>
                <a href="{{ talk.url | relative_url }}" {% unless talk.slides %}style="text-decoration: line-through;"{% endunless %}>{{ talk.title }}</a>
                <span class="talks-toc-event"><small>{{ talk.events[0].event }}</small></span>
            </li>
        {% endfor %}
    </ul>
</nav>
<hr>
<br>
{% for talk in talks_sorted %}
<article>
    {% assign slug = talk.url
    | remove_first: "/talks/"
    | remove: "/" %}
    <a href="{{ talk.url }}">
        <img src="{{ '/assets/img/talks/' | append: slug | append: '.png' | relative_url }}"
             align="right"
             height="230"
             alt="{{ talk.title }}"
             title=""
             style="border: 1px solid rgba(124, 255, 0, 0.2);
                    margin: 10px 0px 40px 40px">
    </a>
    <h2>
        <a href="{{ talk.url }}">{{ talk.title }}</a>
    </h2>
    {% for e in talk.events %}
        <h3>{{ e.event }}</h3>
        <time datetime="{{ e.date | date: "%Y-%m-%d" }}">
            {{ e.date | date: "%B %d, %Y" }}
        </time>
        <span>({{ e.location }})</span>
        <br>
        <br>
    {% endfor %}
    <p>
        <em>{{ talk.excerpt | strip_html | truncatewords: 100 }}</em>
    </p>
</article>
<hr>
<br>
{% endfor %}
