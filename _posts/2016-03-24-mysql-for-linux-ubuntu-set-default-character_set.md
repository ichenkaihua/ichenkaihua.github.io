---
layout: post
title: "linux ubuntu下MySQL配置，解决mysql乱码，不能远程连接问题"
description: "mysql for linux ubuntu set default character_set remote connect "
category: linux
tags: [mysql,linux]
permalink: /:year/:month/:day/:title/
---

linux下安装mysql-server后，mysql-server默认编码为`latium`，这样就导致mysql存储中文乱码。mysql为安全起见，设置`root`账户只能`localhost`登陆，导致其他主机不能访问mysql，今天配置云主机的mysql时，碰到了些问题，记录下来，避免再犯。我的系统是`ubuntu 14.04 LTS`，mysql版本为`5.5`，对其他linux系统来说，解决方法也是一样。<!-- more -->

## Mysql乱码问题
mysql安装时，为mysql数据库设置的默认编码为`latin1`，导致我们用mysql存取数据时，显示乱码。

解决方式很简单:

* 打开编辑`/etc/mysql/my.conf`文件，vi/vim:`sudo  vim /etc/my.conf`
* 在`[client]`节点下添加一行配置:`default-character-set=utf8`
* 在`[mysqld]`节点下添加一行配置:`character-set-server = utf8`
* 保存文件,vi/vim:`:wq`
* 重启mysql: `sudo /etc/init.d/mysql restart`

## Mysql不能远程连接问题
为安全起见，Mysql为`root`账户设置了登陆主机为`localhost`,即只允许在本机登陆。`root`账户作为上帝般的角色，当然不能随便给别人。较好的解决办法是，新开一个mysql账户，给特定的权限，允许所有主机登陆。

### 添加Mysql账户

有两种方式添加用户:

*  使用mysql提供的命令`CREATE USER`来
* 用`root`账户手动添加数据到`mysql.user`表中:`insert into mysql.user values(x,x,x,....)`

个人推荐使用第一种方式，简单方便
下面是创建用户`chenkaihua`,并给予所有权限，允许所有主机远程登陆,

```sql
CREATE USER 'chenkaihua'@'%' IDENTIFIED BY 'pass-chenkaihua';
```
然后为`chenkaihua`这个账户设置权限:

```sql
GRANT ALL  ON  *.*  TO 'chenkaihua'@'%'; 
```
具体语法可以参考mysql官方文档。

在创建用户了之后，发现还是不能远程连接。连接时，mysql提示`10061`错误。
出现这个错误，是因为mysql有个配置属性`bind-address= 127.0.0.1`,即只接收本机ip登陆，需要改成接收所有地址登陆(`0.0.0.0`)。

### 允许所有主机登陆

**步骤如下:**

* 打开编辑`/etc/mysql/my.conf`文件，vi/vim:`sudo  vim /etc/my.conf`
* 修改属性 `bind-address= 127.0.0.1` 为 `bind-address= 0.0.0.0`
* 保存文件,vi/vim:`:wq`
* 重启mysql: `sudo /etc/init.d/mysql restart`

## 总结

经过上述配置之后，能正常使用mysql。建议为每一个数据库创建一个用户，便于管理权限，避免不可恢复错误。






