---
title: "Marc's YouTube videos"
permalink: /videos/
entries_layout: grid
layout: collection
output: true
---

Click on the thumbnails to view on YouTube.

{% for post in site.documents %}
    {% if post.videoid %}
<div>
<h2>{{post.videotitle}}</h2>
<article>
<a href="https://www.youtube.com/watch?v={{post.videoid}}"><img src="/assets/images/thumbnails/{{post.videoid}}.jpg">watch on YouTube</a>
</article>
</div>
    {% endif %}
{% endfor %}
