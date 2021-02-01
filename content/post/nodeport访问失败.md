---
title: nodeport访问失败
#slug: chinese-test
date: 2020-12-17
categories:
- k8s
- iptables
- svc
tags:
- k8s
- svc
thumbnailImagePosition: left
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg
---
nodeport访问失败
<!--more-->

# 通常由于iptables问题和selinux问题
解决方式
```bash
setenforce 0
iptables --flush
iptables -tnat --flush
service docker restart
iptables -P FORWARD ACCEPT
```
官方原文：
Disabling SELinux by running setenforce 0 is required to allow containers to access the host filesystem, which is required by pod networks for example. You have to do this until SELinux support is improved in the kubelet.
翻译就是
通过setenforce 0 关闭SELinux是必须的，这样容器才可以访问主机的文件系统，例如pod的网络操作就需要访问主机文件系统。直到kubelet支持SELinux之前你都需要关闭SELinux。

iptables --flush和iptables -tnat --flush是清空iptables的规则，然后重启docker后会重新加入kubernetes的iptables规则，iptables -P FORWARD ACCEPT则允许转发



# 设置service的nodeport以后外部无法访问对应的端口的问题
关于Service 设置为NodePort 模式后导致外部无法访问对应的端口问题，查看下node节点上已经出现了30002端口的监听，确保服务器外部安全组等限制已经放行该端口，按照网上搜到的教程配置基本没看到要配置iptables相关的问题。
然后查阅了不少资料才看到docker 1.13 版本对iptables的规则进行了改动，默认FORWARD 链的规则为DROP ，带来的问题主要一旦DROP后，不同主机间就不能正常通信了（kubernetes的网络使用flannel的情况）。
查看下node节点上的iptables规则：
```bash
[root@k8s-master ~]# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
KUBE-FIREWALL  all  --  0.0.0.0/0            0.0.0.0/0           

Chain FORWARD (policy DROP)
target     prot opt source               destination         
DOCKER-ISOLATION  all  --  0.0.0.0/0            0.0.0.0/0           
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0  
```
　iptables 被kubernetes接管后的规则比较多，仔细看下FORWARD 规则就发现了，policy DROP状态，这就导致了我们直接访问node节点的IP加上端口会无法访问容器，问题出在iptables上就好解决了。
临时解决方案，直接设置FORWARD 全局为ACCEPT：
```bash
[root@k8s-master ~]# iptables -P FORWARD ACCEPT

[root@k8s-master ~]# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
KUBE-FIREWALL  all  --  0.0.0.0/0            0.0.0.0/0           
```
Chain FORWARD (policy ACCEPT)
这样需要在每台node上执行命令，并且重启node后规则失效；
永久解决方案，修改Docker启动参数，在[Service] 区域末尾加上参数：
```bash
[root@k8s-master ~]# vim /usr/lib/systemd/system/docker.service
[Service]
............
ExecStartPost=/sbin/iptables -I FORWARD -s 0.0.0.0/0 -j ACCEPT
[root@k8s-master ~]# systemctl daemon-reload
[root@k8s-master ~]# systemctl restart docker
```
由于修改了systemctl 文件，所以需要重新载入，重启docker 就可以看到iptables规则已经加上ACCEPT all -- 0.0.0.0/0 0.0.0.0/0
```bash
[root@k8s-master ~]#  iptables -L -n
Chain INPUT (policy DROP)
target     prot opt source               destination         
KUBE-FIREWALL  all  --  0.0.0.0/0            0.0.0.0/0           

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0 
```
