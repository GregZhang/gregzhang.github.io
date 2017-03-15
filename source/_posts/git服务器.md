---
title: git服务器
date: 2017-02-23 09:17:35
tags:
  - linux
categories: 
  - 运维
---

# 安装 Git
服务器端：
```bash
#yum install -y git
```
安装完后，查看 Git 版本.
```bash
[root@localhost ~]# git --version
git version 1.7.1
```

客户端：
下载 Git for Windows，地址：https://git-for-windows.github.io/
安装完之后，可以使用 Git Bash 作为命令行客户端。
安装完之后，查看 Git 版本
```bash
$ git --version
git version 2.8.4.windows.1
```
 

# 创建 git 用户 
服务器端创建 git 用户，用来管理 Git 服务，并为 git 用户设置密码
```bash
[root@localhost home]# id git
id: git 无此用户
[root@localhost home]# useradd git
[root@localhost home]# passwd git
```


# 服务器端创建 git 仓库
设置 /home/data/git/gittest.git 为 Git 仓库
然后把 Git 仓库的 owner 修改为 git
```bash
[root@localhost home]# mkdir -p data/git/gittest.git
[root@localhost home]# git init --bare data/git/gittest.git
Initialized empty Git repository in /home/data/git/gittest.git/
[root@localhost home]# cd data/git/
[root@localhost git]# chown -R git:git gittest.git/
```

# 客户端 clone 远程仓库
进入 Git Bash 命令行客户端，创建项目地址（设置在 d:/wamp64/www/gittest_gitbash）并进入:

```bash
dee@Lenovo-PC MINGW64 /d
$ cd wamp64/www
dee@Lenovo-PC MINGW64 /d/wamp64/www
$ mkdir gittest_gitbash
dee@Lenovo-PC MINGW64 /d/wamp64/www
$ cd gittest_gitbash
dee@Lenovo-PC MINGW64 /d/wamp64/www/gittest_gitbash
```

然后从 Linux Git 服务器上 clone 项目：
```bash
$ git clone git@192.168.56.101:/home/data/gittest.git
```