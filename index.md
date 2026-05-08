---
layout: single
author_profile: true
excerpt: "Geospatial automation, spatial SQL, and data analytics. Building efficient, code-driven solutions for complex geographic problems."
header:
  overlay_image: /assets/images/digital_globe.jpg
  overlay_filter: 0.5
  caption: "Data-driven spatial insights"

---

### Experience & Skills
I am a Geospatial Analyst with 10 years of experience in the aerospace and defense sectors. My work focuses on moving away from manual map-making toward automated, code-driven insights.

* **Core Skills:** Python (GeoPandas, Rasterio), SQL (PostGIS, DuckDB), ArcGIS Pro Automation.
* **Focus:** Climate-tech, urban mobility, and environmental analytics.

---

### Featured Projects

<div class="entries-grid">
  {% for project in site.projects limit:3 %}
    {% include archive-single.html type="grid" %}
  {% endfor %}
</div>

[View all projects](/projects-archive/){: .btn .btn--primary}