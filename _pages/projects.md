---
layout: page
title: Projects
permalink: /projects/
description: 
nav: true
nav_order: 2
display_categories:
  - Thesis
  - Projects
toc:
  sidebar: left
---

Here are some of the projects I've worked on, over the years.

{% for category in page.display_categories %}
  {% assign projects_in_category = site.projects | where: "category", category | sort: "importance" %}
  {% if projects_in_category.size > 0 %}

  {% endif %}
{% endfor %}


{% comment %}
Group all items from the `projects` collection by their `category`
and list each project as a link to its own page.
{% endcomment %}

{% assign all_projects = site.projects | sort: "importance" %}
{% assign groups = all_projects | group_by: "category" %}

<ul>
  {% for group in groups %}
    <li>
      {{ group.name }}
      <ul>
        {% for project in group.items %}
          <li>
            <a href="{{ project.url | relative_url }}">
              {{ project.title }}
            </a>
          </li>
        {% endfor %}
      </ul>
    </li>
  {% endfor %}
</ul>
