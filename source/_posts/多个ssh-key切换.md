---
title: 多个ssh-key切换
date: 2017-03-03 15:46:34
tags:
  - git
  - ssh公钥
  - 多个ssh-key
categories: 
  - 软件
---

# 配置项
~/.ssh目录创建config文件，该文件用于配置私钥对应的服务器。
内容如下：
```bash
# Default github user(first@mail.com)
Host github.com
HostName github.com
User git
IdentityFile C:/Users/username/.ssh/id_rsa

# second user(second@mail.com)
Host github-second
HostName github.com
User git
IdentityFile C:/Users/username/.ssh/id_rsa_second
```
Host随意即可，方便自己记忆，后续在添加remote是还需要用到。
配置完成后，在连接非默认帐号的github仓库时，远程库的地址要对应地做一些修改，比如现在添加second帐号下的一个仓库test，则需要这样添加：
```bash
#并非原来的git@github.com:second/test.git
git remote add test git@github-second:second/test.git 
```
这样每次连接都会使用id_rsa_second与服务器进行连接。至此，大功告成！

## 注意
github根据配置文件的user.email来获取github帐号显示author信息，所以对于多帐号用户一定要记得将user.email改为相应的email(second@mail.com)。

## 参考
github帮助文档：
http://help.github.com/win-set-up-git/
http://help.github.com/multiple-ssh-keys/

# 错误说明
## Bad file number
这大多是因为开启了防火墙造成的。可以关闭防火墙后再试，或者尝试Http/Https的方式，一般防火墙都不会禁止这种方式。

## Permission denied
这个原因比较多，比如：
1. 找不到私钥文件
2. 私钥生成有误，比如生成时没指定类型

## Unable to use key file
需要将服务器端生成的私钥的格式转换一下putty才能认。运行PUTTYGEN.EXE，先把ssh-keygen生成的私钥load进去，然后再save private key 就可以了

来源:http://blog.csdn.net/su_dong/article/details/52684202