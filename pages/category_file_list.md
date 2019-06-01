---
layout: page
title: All articles are here
titlebar: category_file_list
subtitle: <span class="mega-octicon octicon-calendar"></span>&nbsp;&nbsp;专题系列： &nbsp;&nbsp; <a href ="http://www.buildupchao.cn/arch.html"><font color="#1A0DAB">架构</font></a>&nbsp;&nbsp; <a href ="http://www.buildupchao.cn/bigdata.html"><font color="#EB9439">大数据</font></a>&nbsp;&nbsp; <a href ="http://www.buildupchao.cn/java.html"><font color="#1E90FF">Java</font></a>
menu: archives
css: ['blog-page.css']
permalink: /category_file_list.html
---

<% var ca = request.getParameter("dire_category") %>
{% include category_file_list.html dire_category=ca %}
