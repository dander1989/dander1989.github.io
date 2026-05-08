---
layout: archive
title: "Project Portfolio"
permalink: /projects-archive/
author_profile: true
---

Debug: Number of projects found: {{ site.projects | size }}

{% for post in site.projects %}
  {% include archive-single.html type="grid" %}
{% endfor %}