---
title: k8s 监控采集信息
#slug: chinese-test
date: 2020-04-10
categories:
- k8s
- prometheus
- hpa
- metrics-server
tags:
- prometheus
- hpa
thumbnailImagePosition: left
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg
---

<!--more-->

自定义指标：

[参考](https://github.com/stefanprodan/k8s-prom-hpa/blob/master/README.md)

metrics-server:

k8s 1.18.0 安装 metrics-server 0.3.6 相关注意事项：
```bash
kube-apiserver.yaml 添加参数
      - --enable-aggregator-routing=true
      - --requestheader-allowed-names=aggregator,front-proxy-client



metrics-server 更新镜像，修改参数
      containers:
      - name: metrics-server
        image: kylincloud2.hub/kube-system/metrics_server:v0.3.6
        command:
          - /metrics-server
          - --metric-resolution=5s
          - --kubelet-preferred-address-types=InternalIP
          - --kubelet-insecure-tls
            #- --requestheader-client-ca-file=/opt/front-proxy-ca.pem  #注释此行
        imagePullPolicy: IfNotPresent

注释所有挂载
```