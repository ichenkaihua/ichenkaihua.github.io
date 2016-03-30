---
layout: post
title: "解决 Choose unique values for the 'webAppRootKey' context-param in your web.xml files! 错误"
description: "tomcat multi project WebAppRootKey ,Choose unique values for the 'webAppRootKey' context-param in your web.xml files!"
category: j2ee
fullwidth: true
tags: [tomcat]
date: 2015-11-22 14:38:19
permalink: /:year/:month/:day/:title/
---

错误:

```
Choose unique values for the 'webAppRootKey' context-param in your web.xml files!
```

这个错误出现在部署多个项目时，表现为只有第一个项目部署成功，其他项目出现404错误。原因在于如果有多个项目，每个项目都需要一个唯一标记，这个标记叫做`webAppRootKey`，默认值为`webapp-root`。因此需要我们注意，在web.xml中尽量配置`webAppRootKey`，避免在部署多项目时出现错误。

**解决方法**:在`web.xml`中配置`webAppRootKey`:

```xml
<context-param>  
    <param-name>webAppRootKey</param-name>  
    <param-value>your-app-key</param-value>  
  </context-param>  
```

