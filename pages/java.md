---
layout: page
title: Java跨阶修炼
titlebar: java
subtitle: <span class="mega-octicon octicon-clippy"></span> &nbsp;&nbsp; Java觉醒之路>&nbsp;&nbsp;>&nbsp;&nbsp;
menu: java
css: ['blog-page.css']
permalink: /java
---

<div class="row">

    <div class="col-md-12">

        <ul id="posts-list">
            {% for post in site.posts %}
                {% if post.category=='Java'
                    or post.category=='java'
                    or post.category=='jvm'
                    or post.category=='JVM'
                    or post.tags contains 'java'
                    or post.tags contains 'Java' %}
                <li class="posts-list-item">
                    <div class="posts-content">
                        <span class="posts-list-meta">{{ post.date | date: "%Y-%m-%d" }}</span>
                        <a class="posts-list-name bubble-float-left" href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
                        <span class='circle'></span>
                    </div>
                </li>
                {% endif %}
            {% endfor %}
        </ul>

        <!-- Pagination -->
        {% include pagination.html %}

        <!-- Comments -->
       <div class="comment">
         {% include comments.html %}
       </div>
    </div>

</div>
<script>
    $(document).ready(function(){

        // Enable bootstrap tooltip
        $("body").tooltip({ selector: '[data-toggle=tooltip]' });

    });
</script>
