---
title: ubuntu
#slug: chinese-test
date: 2020-07-24
categories:
- ubuntu
tags:
- ubuntu
thumbnailImagePosition: left
thumbnailImage: /img/ubuntu.jpeg
---
登录页面去掉多余的用户,更换登录界面背景图,ulimit 修改
<!--more-->

# 登录页面去掉多余的用户

linxu系统用户的 id >= 1001 的用户的账户会出现在登录界面.
把id改小就可以了

# ubuntu18.04如何更换登录界面背景图
https://blog.csdn.net/unicorn_mitnick/article/details/85236456

# ulimit 修改
https://blog.csdn.net/zhulx_sz/article/details/88309367

# 网络

查看不同状态的连接数数量

`netstat -an | awk '/^tcp/ {++y[$NF]} END {for(w in y) print w, y[w]}'`

查看每个ip跟服务器建立的连接数

`netstat -nat|grep "tcp"|awk ' {print$5}'|awk -F : '{print$1}'|sort|uniq -c|sort -rn`

查看每个ip建立的ESTABLISHED/TIME_OUT状态的连接数

`netstat -nat|grep ESTABLISHED|awk '{print$5}'|awk -F : '{print$1}'|sort|uniq -c|sort -rn`

网卡多队列

https://www.cnblogs.com/qinyujie/p/11127660.html

查找僵尸进程

`ps -A -ostat,ppid,pid,cmd |grep -e '^[Zz]'`