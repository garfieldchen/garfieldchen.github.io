---
layout: page
title: Tags
subtitle: Keep Calm and Carry On.
permalink: /tags/
---

<section class="post-tags">
{% assign tags_list = site.tags %}
    
{% if tags_list.first[0] == null %}
    {% for tag in tags_list %}
        <a class="post-tags-item" href="#{{ tag | slugify }}">{{ tag }}</a>
    {% endfor %}
{% else %}
    {% for tag in tags_list %}
        <a class="post-tags-item" href="#{{ tag[0] | slugify }}">{{ tag[0] }}</a>
    {% endfor %}
{% endif %}
    
{% assign tags_list = nil %}

</section>

-----------------

{% for tag in site.tags  %}

## {{ tag[0] | capitalize }}
    
    {% assign pages_list = tag[1] %}

    {% for post in pages_list reversed %}
        {% if post.title != null %}
            {% if group == null or group == post.group %}
        
- [ {{ post.title }} ]( {{ site.url }}{{ post.url | prepend: site.baseurl }} ) <span class="date"><time datetime="{{ post.date | date_to_long_string }}"> - {{ post.date | date_to_long_string }}</time></span>

            {% endif %}
        {% endif %}
    {% endfor %}
    
    {% assign pages_list = nil %}

    {% assign group = nil %}
{% endfor %}
