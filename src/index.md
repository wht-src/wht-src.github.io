---
layout: base.njk
title: WHT's Blogs
---

# WHT's Blogs

Here stores my guides to softwares, mainly Linux and programming.

## Recent Posts

<ul class="post-list">
{%- for post in collections.posts | reverse -%}
  <li>
    <a href="{{ post.url }}">{{ post.data.title }}</a>
    <p class="post-meta">
      <time datetime="{{ post.date | htmlDateString }}">{{ post.date | readableDate }}</time>
    </p>
  </li>
{%- endfor -%}
</ul>
