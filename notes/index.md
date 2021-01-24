---
layout: page
title:  Notes
header: notes
group: navigation

---

{% include JB/setup %}

<ol>
{% include JB/sort_collection collection=site.pages sort_by="title" %}
{% assign pages_list = sort_result %}
{% include JB/notes_list %}
</ol>

