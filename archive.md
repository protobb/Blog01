---
layout: page
title: Archive
permalink: /archive/
---

{% assign old_year=0 %}
{% for post in site.posts %}
{% capture new_year%}} {{post.date | date: '%Y'}} {% endcapture %}
{% if old_year!=new_year %}
### {{post.date | date: '%Y'}}
{% assign old_year=new_year %}
{% endif %}
+ {{post.date | date: '%m/%d'}} [{{post.title}}]({{post.url}}) 
{% endfor %}
