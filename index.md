---
layout: single
author_profile: true
title: ""
excerpt: "Geospatial automation, spatial SQL, and data analytics. Building efficient, code-driven solutions for complex geographic problems."
header:
  overlay_image: /assets/images/digital_globe.jpg
  overlay_text: "David Anderson"
  overlay_filter: 0.5
  caption: "Geospatial Data Analyst | Applied Scientist"

---

### Experience & Skills
I am a Geospatial Analyst with 10 years of experience with geospatial methods and tools. I'm working on focusing away from just static mapmaking and more towards automated, code-driven insights using cloud-native tools and data formats. For more information see my 'About' section!

I would love to connect with you on LinkedIn. Send me a connection request and let's talk!

Also be sure to check out my personal projects I have been working on!

| Domain | Tools & Technologies |
| :--- | :--- |
| **Programming** | Python (Pandas / GeoPandas, Rasterio, ArcPy), SQL (PostgreSQL, PostGIS, DuckDB, SedonaDB) |
| **Analysis** | Spatial Statistics, Viewshed Analysis, DEM and LiDAR Processing |
| **Automation** | ArcGIS Pro SDK, ETL Pipelines |
| **Desktop**  | ArcGIS Pro, QGIS |
| **Cloud** | AWS S3 and CLI, Geoparquet, Cloud Optimized GeoTIFFs (COGs) |

---

### Featured Projects

<div class="entries-grid">
  {% assign sorted_projects = site.projects | sort: "priority" %}
    {% for post in sorted_projects limit:3 %}
      {% include archive-single.html type="grid" %}
    {% endfor %}
    ```
</div>

[View all projects](/projects-archive/){: .btn .btn--primary}