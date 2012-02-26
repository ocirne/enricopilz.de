---
layout: default
title: enricopilz.de
topimage: top00.jpg
---

<div>
<ul class="home">
{% for post in site.posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a> <span style="font-size: small">({{ post.date | date: "%d.%m.%Y" }})</span></li>
{% endfor %}
</ul>
</div>
