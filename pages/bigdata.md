---
layout: page
title: 大数据之路
titlebar: 大数据之路
subtitle: <span class="mega-octicon octicon-keyboard"></span>&nbsp;&nbsp; 我的大数据之路： &nbsp;&nbsp; <a href ="http://www.buildupchao.cn/spark.html"><font color="#1A0DAB">Spark</font></a>&nbsp;&nbsp; <a href ="http://www.buildupchao.cn/hadoop.html"><font color="#EB9439">Hadoop</font></a>&nbsp;&nbsp; <a href ="http://www.buildupchao.cn/bigdata.html"><font color="#1E90FF">大数据</font></a>
menu: arch
css: ['blog-page.css']
permalink: /bigdata
---

<div class="row">

    <div class="col-md-12">

        <ul id="posts-list">
            {% for post in site.posts %}
                {% if post.category=='bigdata'
                  or post.category contains '大数据'
                  or post.category contains 'bigdata'
                  or post.tags contains '大数据'
                  or post.title contains '大数据'
                %}
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
