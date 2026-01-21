---
layout: default
image: "/assets/img/og-image.png"
---

# vavkamil@localhost:~$

Hi 👋,

Welcome to my personal website! My crime is that of curiosity.

> "This is it... this is where I belong..."<br>
> I know everyone here... even if I've never met them, never talked to<br>
> them, may never hear from them again... I know you all...

<hr>
<br>

## Latest [blog]({{ site.baseurl }}/blog) posts & [talks]({{ site.baseurl }}/talks)

<style>
  a { text-decoration: none; }

  .mixed-toc ul { list-style: none; padding-left: 0; }
  .mixed-toc li { display: flex; align-items: baseline; gap: 0.75rem; margin: 0.25rem 0; }
  .mixed-year { width: 4.5rem; color: #7CFF00; flex-shrink: 0; }
  .mixed-right { margin-left: auto; opacity: 0.8; white-space: nowrap; }

  @media (max-width: 640px) {
    .mixed-toc li {
      display: grid;
      grid-template-columns: 4.5rem 1fr;
      column-gap: 0.75rem;
      row-gap: 0.25rem;
      align-items: start;
      min-width: 0;
      padding-bottom: 0.4rem;
    }

    .mixed-year {
      grid-column: 1;
      grid-row: 2;
    }

    .mixed-toc li > small {
      grid-column: 2;
      grid-row: 2;
      justify-self: start;
    }

    .mixed-right {
      grid-column: 2;
      grid-row: 2;
      justify-self: end;
      white-space: nowrap;
      max-width: 100%;
      overflow: hidden;
      text-overflow: ellipsis;
      min-width: 0;
    }

    .mixed-toc li > a {
      grid-column: 1 / -1;
      grid-row: 1;
      min-width: 0;
      overflow-wrap: anywhere;
      display: block;
      padding: 0.15rem 0;
    }
}

</style>

{% assign combined = "" | split: "" %}

{% for post in site.posts %}
  {% capture words %}{{ post.content | number_of_words | minus: 180 }}{% endcapture %}
  {% capture minutes %}
    {% unless words contains "-" %}
      {{ words | plus: 180 | divided_by: 180 }}
    {% endunless %}
  {% endcapture %}

  {%- assign safe_title = post.title | replace: "~~~", "-" -%}
  {%- assign ts = post.date | date: "%s" -%}
  {% capture line %}{{ ts }}~~~{{ post.date | date: "%Y-%m-%d" }}~~~post~~~{{ post.url | relative_url }}~~~{{ safe_title }}~~~{{ minutes | strip }}{% endcapture %}
  {% assign combined = combined | push: line %}
{% endfor %}

{% assign talks_sorted = site.talks | sort: "latest_event_date" %}
{% for talk in talks_sorted %}
  {%- assign safe_title = talk.title | replace: "~~~", "-" -%}
  {% capture right %}{% if talk.events and talk.events.size > 0 %}{{ talk.events[0].event }}{% endif %}{% endcapture %}

  {%- assign has_slides = talk.slides | default: nil -%}
  {%- assign has_slides_flag = has_slides | if: "1", else: "0" -%}

  {%- assign ts = talk.latest_event_date | date: "%s" -%}
  {% capture line %}{{ ts }}~~~{{ talk.latest_event_date | date: "%Y-%m-%d" }}~~~talk~~~{{ talk.url | relative_url }}~~~{{ safe_title }}~~~{{ right | strip }}~~~{% if talk.slides %}1{% else %}0{% endif %}{% endcapture %}
  {% assign combined = combined | push: line %}
{% endfor %}

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
      {% assign ts = parts[0] %}
      {% assign d = parts[1] %}
      {% assign kind = parts[2] %}
      {% assign url = parts[3] %}
      {% assign title = parts[4] %}
      {% assign right = parts[5] %}
      {% assign has_slides = parts[6] | strip %}
      <li>
        <span class="mixed-year">{{ d | date: "%Y-%m" }}</span>
        <small><small><em>{{ kind }}</em></small></small>
        <a href="{{ url }}"
           {% if kind == "talk" and has_slides == "0" %}style="text-decoration: line-through;"{% endif %}>
          {{ title }}
        </a>
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
