---
layout: post
title: "jekyll search via Simple-Jekyll-Search"
description: "jekyll search via Simple-Jekyll-Search | 使用Simple-Jekyll-Search解决jekyll站内搜索问题"
category: web
tags: [jekyll,Simple-Jekyll-Search]
permalink: /:year/:month/:day/:title/
---


这两天换了博客主题，解决了小屏幕不适配问题，也加了些自定义的内容。总之，比较合心意。相对上个博客主题最大变化，就是加入了搜索功能。网络上提供了多种方式解决jekyll搜索的不足，我使用的是[Simple-Jekyll-Search][1]。<!-- more -->

## 常用解决方案
主要有三种方式：

-  google/baidu的站内搜索：优点是嵌入简单，缺点是不稳定。
- 使用jekyll插件，优点是使用简单，但是github-pages不支持插件。
- 使用javascript实现简单静态搜索，*Simple-Jekyll-Search*项目就是这种方式。


## Simple-Jekyll-Search的使用
**Simple-Jekyll-Search**是个开源项目,托管在github:[https://github.com/christian-fei/Simple-Jekyll-Search][1]，方便的话，给个star。

### 下载

```shell
# 下载Simple-jekyll-Search
git clone https://github.com/christian-fei/Simple-Jekyll-Search.git
```

### 配置`search.json`文件
进入博客目录，新建**search.json**文件,内容如下:

```json
{% raw %}
[
  {% for post in site.posts %}
    {
      "title"    : "{{ post.title | escape }}",
      "category" : "{{ post.category }}",
      "tags"     : "{{ post.tags | join: ', ' }}",
      "url"      : "{{ site.baseurl }}{{ post.url }}",
      "date"     : "{{ post.date }}"
    } {% unless forloop.last %},{% endunless %}
  {% endfor %}
]
{% endraw %}
```
**search.json**文件的作用是生成搜索的数据索引，上面的内容能通过标题，tag的关键字搜索，我认为足够了。

### 嵌入模板
找到页面html模板，把js文件引入到html中，然后指定一个搜索框，比如在**default.html**文件中:

```html
<!-- Html Elements for Search -->
<div id="search-container">
<input type="text" id="search-input" placeholder="search...">
<ul id="results-container"></ul>
</div>

<!-- Script pointing to jekyll-search.js -->
<script src="{{ site.baseurl }}/bower_components/simple-jekyll-search/dest/jekyll-search.js" type="text/javascript"></script>
```

### 配置
怎么使用这个搜索呢，非常简单，在**default.html**中使用javascript配置搜索框和`search.json`位置就好了。

```javascript
    SimpleJekyllSearch({
        searchInput: document.getElementById('search-input'),
        resultsContainer: document.getElementById('results-container'),
        json: '/search.json',
        searchResultTemplate: '<li><a href="{url}" title="{desc}">{title}</a></li>',
        noResultsText: '没有搜索到文章',
        limit: 20,
        fuzzy: false
      })
```
解释下主要的配置项:

* **searchInput**:输入框，配置之后会给输入框添加监听，只要输入框的值发生变化，搜索结果的容器也发生变化。
* **resultsContainer**:存放搜索结果的容器，一般是`ul`
* **json**:`search.json`的位置
* **searchResultTemplate**:搜索出的每一项怎么渲染，上例是添加一个`li`,元素会添加在`resultsContainer`中

这样配置之后，在输入框输入关键字，`results-container`就会显示搜索到的结果。

### 配合bootstrap
上述教程虽然实现了搜索功能，但是搜索效果不太美观。bootstrap是个优秀的前端框架，提供了常用组件。`Simple-Jekyll-Search`结合bootstrap的下拉菜单`drapdown`组件，比较完美解决搜索外观问题。直接看代码：

```html
 <div class="dropdown navbar-form navbar-right ">
   <input  id="dLabel" name="word" type="text"  aria-haspopup="true" aria-expanded="false" data-toggle="dropdown" class="form-control typeahead"   placeholder="搜索">
  
  <ul class="dropdown-menu" aria-labelledby="dLabel" id="c">
    
  </ul>
</div>
```
这段代码的意思是，只要输入框被点击，下拉菜单就显示，配合**Simple-Jekyll-Search**，实现效果如下:
![jekyll-search-bootstrap](http://7xivpo.com1.z0.glb.clouddn.com/jekyll-search-bootstrap.png)
如果有不明白的地方，查看这篇文章的网页源码就可以看到了。

## 参考资料
> * [Simple-Jekyll-Search-Github][1]

[1]: https://github.com/christian-fei/Simple-Jekyll-Search