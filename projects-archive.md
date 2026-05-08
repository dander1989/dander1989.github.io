---
layout: archive
title: "Project Portfolio"
permalink: /projects-archive/
author_profile: true
---

{% for project in site.projects %}
  {% include archive-single.html type="grid" %}
{% endfor %}