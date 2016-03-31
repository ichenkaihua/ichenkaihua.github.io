---
layout: post
title: "windows下生成ssh证书并避免每次输入密码(ssh-agent)"
description: "windows --ssh-key-and--ssh-agent-usage-via-git"
category: [build tools]
tags: [git,ssh]
date: 2016-03-03 00:07:48
permalink: /:year/:month/:day/:title/
---

windows下使用git时（我使用的是git bash软件），如果使用`https`协议,则每次远程操作都要输入用户名和密码(github/oschina)，既繁琐又费时。如果使用`git ssh`	协议，虽然不用输入帐号密码，每次提交依然需要输入ssh的密钥密码，也是繁琐。`ssh-keygen`用于生成ssh证书，`ssh-agent`用于保存ssh密码。配置好这两个工具后，多次远程操作只需要一次认证。<!-- more -->

## 下载安装git
个人只使用过`git bash`这个软件，其他未使用，不做评论，可以上<http://git-scm.com>查看并下载git。

## 配置git

配置git的email和name，不然不能push，把邮箱和用户名改成你自己的

* 配置email:`git config --global user.email 'xyz@xx.com' `
* 配置用户名: `git config --global user.name 'myname' `


## 生成ssh证书
安装完git之后，就需要生成ssh证书,我的是win10系统，其他系统未测试

**1** 打开`git bash`,输入以下命令,把`your_email@example.com`改成你的邮箱名


``` shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
按照它的提示，确定ssh key存储的位置和设置密码，最终你可以看到`~/.ssh`下新生成了两个文件`id_rsa`、`id_rsa.pub`

**2** 把ssh公钥添加到github/oschina用户账户。
在`git bash`下，`cat .ssh/id_rsa.pub`,然后复制cat出来的内容(也可以用记事本打开)，添加到github或者开源中国的ssh keys，如果公钥对不上，则无权限远程操作。

## 避免每次操作输入ssh密码
上面操作之后，虽然可以用ssh提交，但是每次操作都要输入ssh密码。`ssh-agent`可以解决这位问题。

**1** 在home目录下新建`.bashrc`文件:打开`gi bash`,输入`touch ~/.bashrc`
**2** 用记事本打开`~/.bashrc`,把以下代码复制到文件中，并保存。

``` bat
# Note: ~/.ssh/environment should not be used, as it
#       already has a different purpose in SSH.

env=~/.ssh/agent.env

# Note: Don't bother checking SSH_AGENT_PID. It's not used
#       by SSH itself, and it might even be incorrect
#       (for example, when using agent-forwarding over SSH).

agent_is_running() {
    if [ "$SSH_AUTH_SOCK" ]; then
        # ssh-add returns:
        #   0 = agent running, has keys
        #   1 = agent running, no keys
        #   2 = agent not running
        ssh-add -l >/dev/null 2>&1 || [ $? -eq 1 ]
    else
        false
    fi
}

agent_has_keys() {
    ssh-add -l >/dev/null 2>&1
}

agent_load_env() {
    . "$env" >/dev/null
}

agent_start() {
    (umask 077; ssh-agent >"$env")
    . "$env" >/dev/null
}

if ! agent_is_running; then
    agent_load_env
fi

# if your keys are not stored in ~/.ssh/id_rsa or ~/.ssh/id_dsa, you'll need
# to paste the proper path after ssh-add
if ! agent_is_running; then
    agent_start
    ssh-add
elif ! agent_has_keys; then
    ssh-add
fi

unset env
```
**3** 关闭并重新打开`git bash`,输入ssh密码，以后远程操作都不需要输入密码了。


