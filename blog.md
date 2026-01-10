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

<br>

## Archive & external old stuff

<ul>
    <small><em>
        <li>2022-02-27 - <a target="_blank"
            href="https://tunasec.com/blog/kyberneticka-valka-behem-utoku-na-ukrajinu/">
            Kybernetická válka během útoku na Ukrajinu</a></li>
        <!-- https://tunasec.com/blog/kyberneticka-valka-behem-utoku-na-ukrajinu/ -->
        <li>2021-10-05 - Pi-hole ~ blokace reklam a sledování uživatelů na úrovni DNS</li>
        <!-- https://tunasec.com/blog/pi-hole-blokace-reklam-sledovani-dns/ -->
        <li>2021-06-23 - Základní bezpečnostní HTTP hlavičky pro váš web</li>
        <!-- https://tunasec.com/blog/zakladni-bezpecnostni-http-hlavicky-pro-vas-web/ -->
        <li>2021-03-14 - Přidejte si na web soubor security.txt</li>
        <!-- https://tunasec.com/blog/pridejte-si-na-web-soubor-security-txt/ -->
        <li>2014-12-29 - Fake Cell Phone Towers & Stealth SMS</li>
        <!-- https://www.soom.cz/clanky/1163--Fake-Cell-Phone-Towers-Stealth-SMS -->
        <li>2014-08-09 - Jak na vlastní DNS server</li>
        <!-- https://www.soom.cz/clanky/1155--Jak-na-vlastni-DNS-server -->
        <li>2014-04-04 - Google Nexus 7 Pwn Pad - Kali Linux</li>
        <!-- https://www.soom.cz/clanky/1142--Google-Nexus-7-Pwn-Pad-Kali-Linux -->
        <li>2013-08-07 - Forenzní analýza: identifikace tiskárny</li>
        <!-- https://www.soom.cz/clanky/1118--Forenzni-analyza-identifikace-tiskarny -->
        <li>2013-07-19 - Android - úvod do security</li>
        <!-- https://www.soom.cz/clanky/1116--Android-uvod-do-security -->
        <li>2013-03-08 - Bezpečnost rodných čísel</li>
        <!-- https://www.soom.cz/clanky/1095--Bezpecnost-rodnych-cisel -->
        <li>2010-01-18 - Rooting pro zacatecniky</li>
        <!-- https://web.archive.org/web/20101130005013/https://security-portal.cz/blog/rooting-pro-zacatecniky -->
        <li>2009-08-03 - 50 nejpoužívanějších hesel pro web v ČR</li>
        <!-- https://web.archive.org/web/20090823060027/https://security-portal.cz/clanky/50-nejpou%C5%BE%C3%ADvan%C4%9Bj%C5%A1%C3%ADch-hesel-pro-web-v-%C4%8Dr -->
        <li>2007-12-31 - Robert Trappan Morris</li>
        <!-- https://www.soom.cz/clanky/761--Robert-Trappan-Morris -->
        <li>2007-07-24 - Loyd Blankenship</li>
        <!-- https://www.soom.cz/clanky/750--Loyd-Blankenship -->
        <li>2007-05-31 - Gary McKinnon</li>
        <!-- https://www.soom.cz/clanky/747--Gary-McKinnon -->
        <li>2007-05-22 - David L. Smith</li>
        <!-- https://www.soom.cz/clanky/985--David-L-Smith -->
        <li>2007-05-21 - MafiaBoy</li>
        <!-- https://www.soom.cz/clanky/984--MafiaBoy -->
        <li>2007-04-14 - LeetSpeak!</li>
        <!-- https://www.soom.cz/clanky/976--LeetSpeak -->
    </em></small>
</ul>

