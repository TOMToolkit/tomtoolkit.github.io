---
layout: default
---

# Target Observation Manager Toolkit - News

<ul class="post-list">
  {% for post in site.posts %}
    <li>
      {{ post.date | date: '%d %b %y' }} <a href="{{ post.url }}"> {{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

