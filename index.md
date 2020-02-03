---
layout: splash
permalink: /
header:
  overlay_color: "#5e616c"
  overlay_image: /assets/images/teaser.jpg
  sidebar:
    nav: "docs"
excerpt: >
  <br /><br /><br />
feature_row:
  - image_path: /assets/images/bigdata.jpg
    title: "Data"
    excerpt: "R, Python, Hadoop, Spark, Kafka, Workflow, Analysis, Engineering"
  - title: "Development"
    image_path: /assets/images/software.png
    excerpt: "Python, R, Cloud, DevOps, Web, Infra"
  - title: "Life"
    image_path: /assets/images/travel.jpg
    excerpt: "Travel, Soul, Food, Art"

---
{% include feature_row %}

{{ content }}

<h3 class="archive__subtitle">{{ site.data.ui-text[site.locale].recent_posts | default: "Recent Posts" }}</h3>


{% if paginator %}
  {% assign posts = paginator.posts %}
{% else %}
  {% assign posts = site.posts %}
{% endif %}

{% for post in posts %}
  {% include archive-single.html %}
{% endfor %}

{% include paginator.html %}
