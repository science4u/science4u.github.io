---
layout: page
subheadline: "Projects"
title: "Projects!"
teaser: "These are some projects we want to share. <em>Feeling Responsive</em> uses <a href='http://srobbin.com/jquery-plugins/backstretch/'>Backstretch by Scott Robin</a> to expand them from left to right. The width should be 1600 pixel or higher using a ratio like 16:9 or 21:9 or 2:1."
header:
   image_fullwidth: "header_unsplash_5.jpg"
permalink: "/projects/"
---
<ul>
    {% for post in site.tags.header %}
    <li><a href="{{ site.url }}{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>