---
layout: archive
permalink: /visualizations/
title: "Visualization Posts"
author_profile: true
classes: wide
header:
  image: "/assets/images/island.jpeg"
---


<div>

  <ul class="post-list">

    {% capture tag %}{{ page.title | slugify }}{% endcapture %}

    {% for post in site.posts %}
      {% if post.tags contains "Visualization" %}
        {% include archive-single-mod.html  %}

      {% endif %}
    {% endfor %}

  </ul>

</div>

{% include paginator.html %}
