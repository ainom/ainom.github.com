---
layout: page
title: Ainom
tagline: 【Ai Natural Organic Matter】【爱码农-更爱天然有机物】
---
{% include JB/setup %}

</br>
</br>
</br>

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

<div id="cz_display">
{% assign posts_all = site.posts %}
{% assign count = 10 %}
{% include custom/posts_all %}
<input type="hidden" id="cz_offset" value="10" />
</div>




</br>
</br>
</br>
## End
If you have any question? you can emaill to me.<a href="mailto:javamickey@163.com">javamickey@163.com</a>



