---
title: k8s 端口使用情况
#slug: chinese-test
date: 2021-08-20
categories:
- k8s
- firewalld
- ufw
tags:
- ufw
thumbnailImagePosition: left
thumbnailImage: /img/port.jpeg
---
k8s 开启防火墙时的端口问题
<!--more-->


# Port

k8s组件、附件常用默认端口如下：

| 组件 | 端口 | 
| :---: | :---: |
| etcd | 2379,2380 |
| kube-apiserver | 6443 |
| kube-controller-manager | 10252,10257 |
| kube-scheduler | 10251,10259 |
| kubelet | 2144,10248,10250,10255 |
| kube-proxy | 10249,10256,30000-32767 |
| calico | 179,9090,9091 |
| docker | 4243 |
| coredns | 53 |
| harbor | 8080,5432,6379,5000,5001 |
| nfs-server | 111,2049 |
| ntp | 123 |
| istio | 9876，15000，15010，15012，15014，15017，15053，8080，15021，80，443，15020，15090 |
| thanos | 9090,10901,10902 |
| prometheus | 9090 |
| alertmanager | 9093 |
| metrics-server | 443 |
| kube-state-metrics | 8080,8081 |



注：开启防火墙后，需放行以上端口，还需放行sts的端口(比如harbor，harbor-database.harbor-redis都是sts，需要放行5432,6379,harbor-core才能正常连接数据库)，