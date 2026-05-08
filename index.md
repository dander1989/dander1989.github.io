---
layout: home
author_profile: true
header:
  overlay_image: /assets/images/banner.jpg
excerpt: "Welcome to my portfolio. Below you will find my latest personal projects I've been working on in GIS and Data Analytics."
---

### Featured Projects

{% for project in site.projects %}
  {% include archive-single.html type="grid" %}
{% endfor %}
