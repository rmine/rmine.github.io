---
layout: page
title: Archives
permalink: /archives/
feature-img: "img/sample_feature_img.png"
---

{% for post in site.posts %}
  {% capture year %}{{ post.date | date: "%Y" }}{% endcapture %}
  {% capture month %}{{ post.date | date: "%B" }}{% endcapture %}
  {% capture nyear %}{{ post.previous.date | date: "%Y" }}{% endcapture %}
  {% capture nmonth %}{{ post.previous.date | date: "%B" }}{% endcapture %}


  {% if forloop.first %}
  <h2>Year {{ nyear }}</h2>
  {% endif %}


* {{ post.date | date: "%Y-%m-%d" }} >> [{{ post.title }}]({{ post.url }})


{% if forloop.last %}
{% else %}
  {% if year != nyear %}
    ---
    <h2>Year {{ nyear }}</h2>
  {% endif %}
{% endif %}


{% endfor %}


