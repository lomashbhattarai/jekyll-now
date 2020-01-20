---
layout: page
title: 
permalink: /topics/
---

A list of topics I would like to write about or have in mind. 

*(as a rule, I can only add topics and never remove it)*

{% for topic in site.data.topics  %}
  - {{topic}}
{% endfor %}
