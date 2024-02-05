---
# layout: projectsLayout
layout: page 
permalink: /projects/
title: My WorkStation
subtitle: Some of my works that might intrigue you!
---

<h1>Kuch to dikhai de na bhai...</h1>

ina mina dika... live reload

{% for project in site.projects %}
  ### {{project.title}}
  [Read more]({{project.url}})
{% endfor %}


##### footer