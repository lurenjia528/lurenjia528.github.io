---
title: docker挂载文件与目录
slug: chinese-test
date: 2020-08-16
categories:
- ubuntu
- docker
tags:
- docker
thumbnailImagePosition: left
thumbnailImage: /img/docker.jpg
coverImage: /img/docker-long.jpg
---
docker挂载文件与目录的不同之处
<!--more-->

解决docker通过volumes挂载文件不生效，修改后容器内数据不同步，需要重启容器才能同步的问题
问题
在宿主机修改文件之后，容器内部不会同步生效，没有发生对应的修改，需要重启容器才可以正常同步。

原因
这是由于linux系统文件挂载机制导致的。
docker通过 volumes 挂载文件到容器中，有两种方式。挂载目录和挂载具体文件。
docker挂载文件时，并不是挂载了某个文件的路径，而是挂载了对应的文件，即挂载了linux指定的inode文件。

当使用vim之类的编辑器进行保存时，它不是直接保存文件，而是采用了备份、替换的策略，就是编辑时，是创建一个新的文件，
在保存的时候，把备份文件替换源文件，这个时候文件的 inode 就发生了变化，而原来 inode 对应的文件其实并没有修改，
也就是容器内的文件没有变化。当重启容器的时候，会挂载新的 inode

inode 示例，修改前：
```bash
root@ubuntu:~/Desktop/fileTest$ stat test.conf 
  文件：test.conf
  大小：7         	块：8          IO 块：4096   普通文件
设备：801h/2049d	Inode：416481      硬链接：1
权限：(0664/-rw-rw-r--)  Uid：( 1000/    root)   Gid：( 1000/    root)
最近访问：2020-03-29 21:45:00.355138304 +0800
最近更改：2020-03-29 21:45:00.359137009 +0800
最近改动：2020-03-29 21:45:00.367134421 +0800
创建时间：-
```
通过 vi 等编辑器进行修改之后，可以发现 inode 的值变了
```bash
root@ubuntu:~/Desktop/fileTest$ stat test.conf 
  文件：test.conf
  大小：7         	块：8          IO 块：4096   普通文件
设备：801h/2049d	Inode：416483      硬链接：1
权限：(0664/-rw-rw-r--)  Uid：( 1000/    root)   Gid：( 1000/    root)
最近访问：2020-03-29 21:45:42.594603856 +0800
最近更改：2020-03-29 21:45:42.594603856 +0800
最近改动：2020-03-29 21:45:42.598604612 +0800
创建时间：-
```
解决
（1）避免直接挂载文件，而是挂载目录，挂载目录不会有上述情况
如果真要挂载目录，那么要将文件权限修改成777，即 -rwxrwxrwx
例如：chmod 777 test.conf

（2）宿主机和容器内一直使用echo想文件中写数据，不通过vi/vim
此时inode不变，文件可同步
