---
layout: post
title: "解决linux ubuntu14 使用teamviewer11 中文乱码问题"
description: "Solve the problem of Linux ubuntu14.04 use teamviewer Chinese garbled"
category: linux
fullwidth: true
tags: [ubuntu,linux,teamviewer,乱码]
date: 2015-12-21 12:45:28
permalink: /:year/:month/:day/:title/
---
Teamviewer是优秀的远程控制软件，支持文件传输，视频录制，聊天通讯等功能。最重要的是支持主流操作系统，甚至android app也可以使用。但是在linux系统使用时，却经常出现中文乱码。linux版的teamview是由windows版wine出来的，但是没有完善字体(可能版权原因)。导致找不到字体，所以乱码了<!-- more -->

乱码效果如下:
![乱码](http://7xivpo.com1.z0.glb.clouddn.com/teamviewer_garbled.png)

**出现原因**:字体缺失，teamviewer使用的是宋体字体，但是在teamviewer字体目录下没有这个字体

**解决方法**

1 windows上或者网络上下载宋体字体(`simsun.ttc `,`simsun.ttf`)

```shell
wget http://down5.pc6.com/qd3/simsun.zip
unzip simsun.zip
```
2.把字体复制到`/opt/teamviewer/tv_bin/wine/share/wine/fonts`目录

```shell
 sudo cp simsun.tt* /opt/teamviewer/tv_bin/wine/share/wine/fonts/
```

3.重启teamviewer

最终效果如下：
![teamviewer_normal](http://7xivpo.com1.z0.glb.clouddn.com/teamviewer_normal.png)


