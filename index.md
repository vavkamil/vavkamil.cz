---
layout: default
---

# vavkamil@localhost:~$

Hi 👋,

Welcome to my personal website! My crime is that of curiosity.

> "This is it... this is where I belong..."<br>
> I know everyone here... even if I've never met them, never talked to<br>
> them, may never hear from them again... I know you all...

<hr>
<br>

## Latest [blog]({{ site.baseurl }}/blog) posts & public [talks]({{ site.baseurl }}/talks)

<style>
  a { text-decoration: none; }

  .mixed-toc ul { list-style: none; padding-left: 0; }
  .mixed-toc li { display: flex; align-items: baseline; gap: 0.75rem; margin: 0.25rem 0; }
  .mixed-year { width: 4.5rem; color: #7CFF00; flex-shrink: 0; }
  .mixed-right { margin-left: auto; opacity: 0.8; white-space: nowrap; }
</style>

{% assign combined = "" | split: "" %}

{%- comment -%} Add blog posts {%- endcomment -%}
{% for post in site.posts %}
  {% capture words %}{{ post.content | number_of_words | minus: 180 }}{% endcapture %}
  {% capture minutes %}
    {% unless words contains "-" %}
      {{ words | plus: 180 | divided_by: 180 }}
    {% endunless %}
  {% endcapture %}

  {%- assign safe_title = post.title | replace: "~~~", "-" -%}
  {%- comment -%}
  Build a sortable line:
  YYYY-MM-DD~~~kind~~~url~~~title~~~right
  {%- endcomment -%}
  {% capture line %}{{ post.date | date: "%Y-%m-%d" }}~~~post~~~{{ post.url | relative_url }}~~~{{ safe_title }}~~~{{ minutes | strip }}{% endcapture %}
  {% assign combined = combined | push: line %}
{% endfor %}

{%- comment -%} Add talks (sorted by latest_event_date) {%- endcomment -%}
{% assign talks_sorted = site.talks | sort: "latest_event_date" %}
{% for talk in talks_sorted %}
  {%- assign safe_title = talk.title | replace: "~~~", "-" -%}
  {% capture right %}{{ talk.events[0].event }}{% endcapture %}
  {% capture line %}{{ talk.latest_event_date | date: "%Y-%m-%d" }}~~~talk~~~{{ talk.url | relative_url }}~~~{{ safe_title }}~~~{{ right | strip }}{% endcapture %}
  {% assign combined = combined | push: line %}
{% endfor %}

{%- comment -%} Sort newest-first {%- endcomment -%}
{% assign combined = combined | sort | reverse %}

<style>
li {
    border-bottom: 1px dotted rgba(124, 255, 0, 0.2);
}
</style>
<nav class="mixed-toc" aria-label="Latest content">
  <ul>
    {% for row in combined limit: 99 %}
      {% assign parts = row | split: "~~~" %}
      {% assign d = parts[0] %}
      {% assign kind = parts[1] %}
      {% assign url = parts[2] %}
      {% assign title = parts[3] %}
      {% assign right = parts[4] %}

      <li>
        <span class="mixed-year">{{ d | date: "%Y-%m" }}</span>
        <small><small><em>{{ kind }}</em></small></small><a href="{{ url }}">{{ title }}</a>

        <span class="mixed-right">
          <small>
            {% if kind == "post" %}
              {% if right != "" %}{{ right }} minutes{% endif %}
            {% else %}
              {{ right }}
            {% endif %}
          </small>
        </span>
      </li>
    {% endfor %}
  </ul>
</nav>
