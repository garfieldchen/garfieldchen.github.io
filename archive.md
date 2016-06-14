---
layout: page
title: Archive
permalink: /archive/
---

{% assign is_first = true %}
	{% for post in site.posts %}

		{% capture this_year %}
			{{ post.date | date: "%Y" }}
		{% endcapture %}
		
		{% if is_first %}
# {{ this_year }}
		{% endif %}

		{% capture next_year %}
			{{ post.previous.date | date: "%Y" }}
		{% endcapture %}
	
		{% if this_year == next_year %}
			{% assign is_first = false %}
		{% else %}
	
		{% assign is_first = true %}
	{% endif %}

<ul>
		<li><a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a><span class="date"> - {{ post.date | date_to_long_string }}</span>
		</li>
	</ul>
		  
{% endfor %}
