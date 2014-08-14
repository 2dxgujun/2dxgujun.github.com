---
layout: default
---

<div>
  <ul class="listing">
  {% for post in site.posts limit: 1 %}
  <article class="content">
    <section class="title">
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    </section>
    <section class="meta">
    <span class="time">
      <time datetime="{{ post.date | date:"%Y-%m-%d %H:%M:%S" }}">{{ post.date | date:"%Y-%m-%d %H:%M:%S" }}</time>
    </span>
    {% if post.tags %}
    <span class="tags">
      {% for tag in post.tags %}
      <a href="/tags/#{{ tag }}" title="{{ tag }}">#{{ tag }}</a>
      {% endfor %}
    </span>
    {% endif %}
    </section>
    <section class="post">
    {{ post.content }}
    </section>
    </article>
  {% endfor %}
  </ul>

  <div class="divider"></div>
  <ul class="listing main-listing">
    <li class="listing-seperator">今年早些时候的文章</li>
    {% capture year %}{{ site.time | date:"%Y"}}{% endcapture %}
    {% for post in site.posts offset:1 %}
    {% capture y %}{{ post.date | date:"%Y"}}{% endcapture %}
    {% if year != y %}
    {% break %}
    {% endif %}
    <li class="listing-item">
      <time datetime="{{ post.date | date:"%Y-%m-%d %H:%M:%S" }}">{{ post.date | date:"%Y-%m-%d %H:%M:%S" }}</time>
      <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
    </li>
    {% endfor %}
    <li class="listing-seperator"><a href="/archive">更早的文章...</a></li>
  </ul>
</div>
