---
layout: page
title: Development
# subtitle: From the pexels folder
permalink: /development/
---
category: {{page.category}}
title: {{page.title}}

<ul class="posts-list">
  {% for post in site.categories[page.title] %}
    <li>
      {{post.title}}
    </li>
  {% endfor %}
</ul>

<div class="posts">
  {% for post in site.categories[page.title] %}
  <div class="post-teaser">
    {% if post.thumbnail %}
    <div class="post-img">
      <a aria-label="{{ post.title }}" href="{{ post.url | relative_url }}">
        <img alt="{{ post.title }}" src="{{ post.thumbnail | relative_url }}">
      </a>
    </div>
    {% endif %}
    <span>
      <header>
        <h1>
          <a aria-label="{{ post.title }}" class="post-link" href="{{ post.url | relative_url }}">
            {{ post.title }}
          </a>
        </h1>
        {% include blog/post_info.html author=post.author date=post.date %}
      </header>
      {% if site.excerpt or site.theme_settings.excerpt %}
      <div class="excerpt">
        {% if site.excerpt == "truncate" %}
        {{ post.content | strip_html | truncate: '250' | escape }}
        {% else %}
        {{ post.excerpt | strip_html | escape }}
        {% endif %}
      </div>
      {% endif %}
    </span>
  </div>
  {% endfor %}
</div>