---
layout: page
title: Categories
permalink: /categories/
feature-img: "img/category_header.jpg"
---


{% capture site_categories %}{% for category in site.categories %}{{ category | first }}{% unless forloop.last %},{% endunless %}{% endfor %}{% endcapture %}
<!-- site_categories: {{ site_categories }} -->
{% assign category_words = site_categories | split:',' | sort %}
<!-- category_words: {{ category_words }} -->

{% for item in (0..site.categories.size) %}{% unless forloop.last %}
{% capture this_word %}{{ category_words[item] | strip_newlines }}{% endcapture %}
<h2>{{ this_word | cgi_escape }} <small>({{ site.categories[this_word].size }})</small> </h2>
  {% for post in site.categories[this_word] %}
  {% if post.title != null %}
* {{ post.date | date: "%Y-%m-%d" }} >> [{{ post.title }}]({{ post.url }})
  {% endif %}
  {% endfor %}

{% endunless %}{% endfor %}

