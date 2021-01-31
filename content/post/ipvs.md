---
title: ipvs
slug: chinese-test
date: 2020-01-21
categories:
- linux
- ipvs
tags:
- ipvs
thumbnailImagePosition: left
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg
---
ipvs系统参数调整
<!--more-->

1. 修改相关的/etc/sysctl.conf的参数
```bash
kernel.core_pattern = /tmp/core-%p-%e-%t
vm.max_map_count = 655360
net.ipv4.ip_forward = 1
net.ipv4.ip_local_port_range = 2000 65535
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 120
net.ipv4.tcp_max_tw_buckets = 10000
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_mem=94500000 915000000 927000000
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 65536 16777216
net.ipv4.tcp_thin_dupack=1
net.ipv4.tcp_thin_linear_timeouts=1
#net.ipv4.vs.conn_reuse_mode = 1
net.ipv4.vs.conntrack = 1
net.ipv4.vs.expire_nodest_conn = 1
net.netfilter.nf_conntrack_max=2048000
net.netfilter.nf_conntrack_tcp_timeout_established=600
net.core.netdev_max_backlog=102400
net.core.somaxconn=65536
net.core.rmem_default=8388608
net.core.wmem_default=8388608
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.unix.max_dgram_qlen=30000
kernel.panic=1
kernel.core_pattern=core_%e
kernel.sysrq=0
kernel.msgmax=65535
kernel.msgmnb=65535
kernel.shmmax=30923764530
kernel.shmall=7549746
vm.panic_on_oom=1
vm.min_free_kbytes=1048576
vm.swappiness=0
fs.inotify.max_user_watches=8192000
fs.aio-max-nr=1048576
fs.file-max=1048575
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 0
net.ipv4.conf.all.rp_filter = 0
```

2. ipvs参数

``` bash 
echo 'options ip_vs conn_tab_bits=20' >  /etc/modprobe.d/ipvs.conf
reboot
```

3. `apt-get install irqbalance`
