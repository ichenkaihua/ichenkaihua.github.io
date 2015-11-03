---
layout: page
title: 陈开华博客
tagline: java开发者
---
{% include JB/setup %}

{% for post in site.posts %}
<div class = "card">
		<div  class = "date_label">
			<div class="day_month">
      			{{ post.date | date:"%m/%d" }}
      			</div>
      			<div class="year">
      			{{ post.date | date:"%Y" }}
      			</div>
      		</div> 
      		
      		<h1> {{post.title}}</h1>
      		
		{{ post.content  | | split:'<!--break-->' | first }}
	<div class = "read_more">
		<a class="fa fa-link" href="{{ BASE_PATH }}{{ post.url }}">  查看全文&hellip;</a>
	</div>
	
</div>

{% endfor %}

