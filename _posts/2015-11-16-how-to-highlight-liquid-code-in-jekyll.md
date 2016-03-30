---
layout: post
title: "jekyll中高亮Liquid代码"
description: "How to highlight Liquid code in jekyll"
category: web
tags: [jekyll,liquid]
permalink: /:year/:month/:day/:title/
---


Jekyll 使用**Liquid** 模板语言供用户调用。jekyll在生成静态页面时，优先处理liquid语法，即把liquid模板的值替换模板变量，比如{% raw %}`{{ site.title }}`{% endraw %}会替换成`_config`里的`title`值。这样就产生一个问题，有时需要代码高亮liquid语法，如果像平常高亮java语法一样处理，liquid语法变量会被赋值。比如我要高亮{% raw %}`url:{{ site.title }}`{% endraw %}，结果却高亮成了`url:陈开华博客`。Liquid考虑到这种情况，使用{% raw %}`{% raw %}`{% endraw %}标签处理替换问题。<!-- more -->

下面是我要高亮的代码

```
{% raw %}
title:{{ site.title }}
githubname:{{ site.githubname }}
{% endraw %}
```

如果我这样写	:

```
{% raw %}
 ```yaml
title:{{ site.title }}
githubname:{{ site.githubname }}
  ```
{% endraw %}
```

效果是这样的:

```
 ```yaml
title:{{ site.title }}
githubname:{{ site.githubname }}
  ```
```

正确的写法是:
![liquid-highlight](http://7xivpo.com1.z0.glb.clouddn.com/liquid-highlight.png)

效果是:

```
{% raw %}
title:{{ site.title }}
githubname:{{ site.githubname }}
{% endraw %}
```

