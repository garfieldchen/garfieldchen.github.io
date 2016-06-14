---
layout: page
title: Category
subtitle: Do something you like.
permalink: /category/
---

{% for category in site.categories %}
{% capture cat %}{{ category | first }}{% endcapture %}

## {{ cat | capitalize }}
	{% for post in site.categories[cat] %}
- [ {{ post.title }} ]( {{ post.url | prepend: site.baseurl }} )
	     <span class="date"> - {{ post.date | date_to_long_string }}</span>
	{% endfor %}

{% endfor %}
