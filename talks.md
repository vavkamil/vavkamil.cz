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
                <a href="{{ talk.url | relative_url }}">{{ talk.title }}</a>
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
