---
layout: post
title: "linux下获取硬盘容量使用情况，开机挂载分区"
description: "linux下获取硬盘容量使用情况，开机挂载分区-陈开华博客"
category: linux
tags: [linux命令]
date: 2015-11-05 13:57:52
permalink: /:year/:month/:day/:title/
---


使用linux时经常需要获取硬盘分区信息、容量大小，文件大小等信息。linux提供了`df` `du`等命令提供上述信息。经常在linux下需要开机挂载硬盘分区，linux提供了简单的解决方法。
<!-- more -->

### 查看硬盘使用情况
主要有两个命令:`df`,`du`，一个获取分区使用情况，一个获取目录、文件的使用情况。

* **df** :获取已经挂载了的分区使用情况，可以显示总空间、剩余空间、使用百分比、挂载点等信息。
* **du** :获取目录或文件的使用情况，显示大小等信息。

#### df的使用

* 必要参数：
	* `-a` :全部文件系统列表
	 * `-h` :方便阅读方式显示
	* `-H` :等于“-h”，但是计算式，1K=1000，而不是1K=1024
	* `-i`: 显示inode信息
	* `-k`: 区块为1024字节
	* `-l`: 只显示本地文件系统
	* `-m`: 区块为1048576字节
	* `--no-sync` : 忽略 sync 命令
	* `-P`:  输出格式为POSIX
	* `--sync` : 在取得磁盘信息前，先执行sync命令
	* `-T` 文件系统类型
 * 选择参数：
	* `--block-size=<区块大小>`: 指定区块大小
	* `-t<文件系统类型>` : 只显示选定文件系统的磁盘信息
	* `-x<文件系统类型> ` :不显示选定文件系统的磁盘信息
	* `--help` : 显示帮助信息
	* `--version` : 显示版本信息

df常用命令:

```
#获取已挂载分区的使用情况
df -h
```

![df-screen](http://7xivpo.com1.z0.glb.clouddn.com/blkid-screen.png)

#### du的使用

1. 命令格式：
	* `du [选项][文件]`
2.  命令功能：显示每个文件和目录的磁盘使用空间。
3. 命令参数：
	- `-a或-all` : 显示目录中个别文件的大小。   
	- `-b或-bytes` :显示目录或文件大小时，以byte为单位。   
	- `-c或--total`  除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。 
	- `-k或--kilobytes`  以KB(1024bytes)为单位输出。
	- `-m或--megabytes`  以MB为单位输出。   
	- `-s或--summarize`  仅显示总计，只列出最后加总的值。
	- `-h或--human-readable`  以K，M，G为单位，提高信息的可读性。
	- `-x或--one-file-xystem`  以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。 
	- `-L<符号链接>或--dereference<符号链接>` 显示选项中所指定符号链接的源文件大小。   
	- `-S或--separate-dirs`   显示个别目录的大小时，并不含其子目录的大小。 
	- `-X<文件>或--exclude-from=<文件> ` 在<文件>指定目录或文件。   
	- `--exclude=<目录或文件>  `       略过指定的目录或文件。    
	- `-D或--dereference-args`   显示指定符号链接的源文件大小。   
	- `-H或--si ` 与-h参数相同，但是K，M，G是以1000为换算单位。   
	- `-l或--count-links `  重复计算硬件链接的文件。  

du常用命令

```
# 获取当前目录的大小
du -sh
```
![du-screen](http://7xivpo.com1.z0.glb.clouddn.com/du-screen.png)

### 开机挂载硬盘分区

1. 查看硬盘分区:
	- `sudo fdisk  -l`
![fdisk-screen](http://7xivpo.com1.z0.glb.clouddn.com/fdisk.png)
2. 查看硬盘UUID:
	- `sudo blkid`
![blkid-screen](http://7xivpo.com1.z0.glb.clouddn.com/blkid-screen.png)
3. 修改/etc/fstab：
	- `sudo vim /etc/fstab`，后面添加
```
UUID=41baef7a-70fa-4bd0-8ea0-25be9c5ef643(要挂载的硬盘如sdb的UUID)   /mnt(要挂载的目录)  ext3(文件类型)  default  0 0
```
比如我的:
![fstab-cat-screen](http://7xivpo.com1.z0.glb.clouddn.com/fstabl-cat.png)


4. 重启

### 参考资料
> * [每天一个linux命令（33）：df 命令-博客园][df-link]{:target="_blank"}
* [每天一个linux命令（34）：du 命令-博客园][du-link]{:target="_blank"}
* [Ubuntu 硬盘自动挂载-Flowers_World-ChinaUnix博客][1]{:target="_blank"}



[df-link]: http://www.cnblogs.com/peida/archive/2012/12/07/2806483.html
[du-link]: http://www.cnblogs.com/peida/archive/2012/12/10/2810755.html
[1]: http://blog.chinaunix.net/uid-30044407-id-4850756.html

