---
layout: page
title: shared notes
permalink: /notes/
nav: true
nav_order: 2
description: Notes I keep while reading papers and working through courses. Shared as-is; most are living documents, so expect a few rough edges.
---

{% assign paper_notes = site.notes | where: "category", "research paper" | sort: "order" %}
{% assign lecture_notes = site.notes | where: "category", "lecture" %}

## research paper

<ul>
  {% for note in paper_notes %}
  <li>
    <a href="{{ note.url | relative_url }}">{{ note.title }}</a>
    {% if note.description %}<br /><small class="text-muted">{{ note.description }}</small>{% endif %}
  </li>
  {% endfor %}
</ul>

## lecture

{% assign courses = lecture_notes | group_by: "course" | sort: "name" %}
{% for course in courses %}

### {{ course.name }}

<ul>
  {% assign sorted_notes = course.items | sort: "order" %}
  {% for note in sorted_notes %}
  <li>
    <a href="{{ note.url | relative_url }}">{{ note.title }}</a>
    {% if note.description %}<br /><small class="text-muted">{{ note.description }}</small>{% endif %}
  </li>
  {% endfor %}
</ul>
{% endfor %}
