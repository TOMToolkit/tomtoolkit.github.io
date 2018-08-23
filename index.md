---
layout: default
---

# Target Observation Manager Toolkit

The TOM Toolkit project was started in early 2018 with the goal of simplifying
the development of next generation software for the rapidly evolving field
of Astronomy. Read more [about TOMs](/about) and the [motiviation](/about)
for them.

Are you looking to run a TOM of your own? The [documentation](/docs/)
is a good place to get started. The source code for the project
is also [available on Github](https://github.com/tomtoolkit).

# News
<ul class="post-list">
  {% for post in site.posts %}
    <li>
      {{ post.date | date: '%d %b %y' }} <a href="{{ post.url }}"> {{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

