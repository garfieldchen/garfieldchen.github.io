---
layout: default
title: Garfield's Blog
---

<style>

.index-content {
    margin-top: 30%;
    margin-bottom: 30%;
    font-size: 18px;
}

</style>

<center>
    <p class="index-content">
    天下风云出我辈，一入江湖岁月催
    </p>
</center>


{% for post in site.posts %}
- {{ post.date | date_to_string }} [{{ post.title }}]({{ post.url }} )
{% endfor %}


