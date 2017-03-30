---
title: Linux下记录所有用户操作的脚本
date: 2017-03-30 15:46:11
tags:
  - linux
  - bash
categories:
  - 运维
---
# 说明

这个脚本是在网上找到的，稍微做了一些修改，可以实现在Linux下所有用户，不管是远程还是本地登陆，在本机的所有操作都会记录下来，并生成包含“用户/IP/时间”的文件存放在指定位置，方便管理员以后查看不同用户都在服务器上干了些什么！目前这个代码只实现了记录用户的操作命令，但是像vi编辑只会记录vi这条命令，但是在文件里所修改的内容无法记录。。。有时间再研究一下！
将下面的代码追加到/etc/profile文件里即可

# 代码

```bash
PS1="`whoami`@`hostname`:"'[$PWD]'
history
USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`
if [ "$USER_IP" = "" ]
then
USER_IP=`hostname`
fi
if [ ! -d /tmp/history ]
then
mkdir /tmp/history
chmod 777 /tmp/history
fi
if [ ! -d /tmp/history/${LOGNAME} ]
then
mkdir /tmp/history/${LOGNAME}
chmod 300 /tmp/history/${LOGNAME}
fi
export HISTSIZE=4096
DT=`date +"%Y%m%d_%H%M%S"`
export HISTFILE="/tmp/history/${LOGNAME}/${USER_IP} history.$DT"
chmod 600 /tmp/history/${LOGNAME}/*history* 2>/dev/null

```

# 待测试

……
