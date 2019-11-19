---
title: 远程git仓库的建立和配置
date: 2017-12-11 17:11:21
categories: 技术相关
tags: [git, gitweb]
---
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/others/gi2.png" width="20%" height="20%">

本文主要介绍如何建立远程 git 仓库以及如何在 gitweb 页面中显示仓库的描述信息，以 Ubuntu 16.04 LTS 操作系统为例。
<!-- more -->

# 安装并配置 gitweb

参考[基于 Apache 服务器的 gitweb 安装和配置](http://blog.cs-tao.cc/2017/10/19/gitweb%E5%AE%89%E8%A3%85%E5%92%8C%E9%85%8D%E7%BD%AE/)

# 新建文件夹

为了能在 gitweb 上查看仓库的信息，建议在 gitweb 的仓库根目录下新建文件夹（仓库根目录在 gitweb 的配置文件"/etc/gitweb.conf"中由"$projectroot"变量指定）

~~~bash
mkdir Test.git
~~~

git 远程库目录建议带上 .git 后缀。

# 更改文件夹权限并切换到用于远程 ssh 连接的用户

因为我们用于连接私有 git 仓库的方法是 ssh 远程连接，我们以有远程 ssh 登录权限的用户 'CSTao' 为例。

~~~bash
chown -R CSTao:CSTao Test.git
su CSTao
~~~

> 注意：本步骤和第一步的顺序可以交换，那么便可以不使用 chown 命令更改文件夹权限，只切换用户即可。这样做的原因是我新建的文件夹所在目录的权限不属于用户 CSTao，以 CSTao 用户新建文件夹会出现权限不足的警告。

# 建立仓库

 ~~~bash
 cd Test.git
 git init --bare
 ~~~
 和建立本地仓库的命令不一样的是，建立远程仓库其实建立了一个裸仓库，也就是不含文件信息，只有 git 的提交记录。

# 配置描述信息

## 修改描述文件

 ~~~bash
 vim description
 ~~~
 写入描述信息即可

## 修改配置文件

 ~~~bash
 vim config
 ~~~
 在原有内容后添加
 ~~~bash
[gitweb]
        owner = CSTao <whucstao@qq.com>
        URL = ssh://CSTao@39.108.171.209:22/home/git/repositories/Test.git
 ~~~
 通过owner指定gitweb中owner的显示内容，通过URL指定gitweb中URL的显示内容，基本格式为"ssh://[ssh登录的用户名]@[host:ssh端口][远程主机中的仓库目录]"

# 将远程裸仓库克隆到本地

 在本地计算机的特定文件夹中执行：
 ~~~bash
 git clone ssh://CSTao@39.108.171.209:22/home/git/repositories/Test.git
 ~~~

# 添加文件、提交更改、推送到远程

这部分内容为 git 的基本操作，不再赘述
