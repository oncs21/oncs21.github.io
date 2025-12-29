---
layout: archive
title: "Short CV"
permalink: /short-cv/
author_profile: true
redirect_from:
  - /resume
---

{% include base_path %}

Education
======
* M.S. in Computer Science, University of Colorado Boulder, May 2026 (expected)
* B.Tech. in Computer Science and Engineering, Vellore Institute of Technology, July 2024

Internships
======
* AI Software Engineer
  * Simple Ticket LLC (Palm City, FL, USA)

* Full Stack Engineer
  * Y STEM & Chess Inc. (Boise, ID, USA)

* Research Software Engineer
  * Yuan Ze University (Taoyuan, Taiwan)
 
* Machine Learning Researcher
  * DRDO (Delhi, India)

* AI Research Assistant
  * Vellore Institute of Technology (Chennai, Tamil Nadu, India)

Publications
======
  <ul>{% for post in site.publications reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>
  
Teaching
======
  <ul>{% for post in site.teaching reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>
