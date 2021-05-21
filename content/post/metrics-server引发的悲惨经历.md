---
title: metrics-server引发的悲惨经历
#slug: chinese-test
date: 2021-05-21
categories:
- k8s
- prometheus
- hpa
- metrics-server
tags:
- prometheus
- hpa
- metrics-server
thumbnailImagePosition: left
thumbnailImage: /img/hashiqi.png
---

<!--more-->

自定义指标(与本文无关，重点是下面～)：

[参考](https://github.com/stefanprodan/k8s-prom-hpa/blob/master/README.md)


背景：

    开始公司使用的服务器是ft1500a和ft2000+,操作系统是kylin-sp2,后来有个项目，要使用虚拟机和kylin-v10操作系统，
    硬件方面都是arm架构，无非一个是基于ubuntu，一个是基于centos，二进制、镜像都没有啥差别，只是前期的一些安装包需要换，
    deb换成rpm，安装k8s就使用的sp2的安装包(之前做的，在多个现场也用的这个)，在虚拟机V10系统上装完之后，奇怪的事情发生了...

版本说明及报错信息：

  - sp2的包，最开始k8s-1.14.2,metrics-server-0.3.2,后来升级到k8s1.18.10,metrics-server没动，还是0.3.2
  - sp2的包，在1500a和2000+的sp2系统上仍然能正常部署，`kubectl top no` 也能用
  - 换到v10上，能正常安装，但是`kubectl top no`用不了(
    `ServiceUnavailable:the server is currently unable to handle the request(get nodes.metrics.k8s.io)`)，
    metrics-server报错(`x509:certificate signed by unknown authority`)，证书不可用,
    kube-apiserver报错(`
    v1beta1.metrics.k8s.io failed with:failing or missing response from https://10.10.113.25:443/apis/metrics.k8s.io/v1beta1:bad status from https://10.10.113.25:443/apis/metrics.k8s.io/v1beta1:401`
    )

开始修复：

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

按上面命令修改之后，
需要重启apiserver，此时，pod会创建失败(`networkplugin cni unauthorization...`),

此时，每台机器执行`rm -rf /etc/cni/net.d/*`，
清除cni配置，然后，`kubectl delete -f calico.yaml`,再`kubectl apply -f calico.yaml`
重启calico，然后，kube-system命名空间下，哪个服务起不来，就delete 再apply,然后，kube-system命名空间下的服务正常了，

接着，发现凡是注入istio-sidecar的pod都起不来(`
error	citadelclient	Failed to create certificate: rpc error: code = Unauthenticated desc = request authenticate failure
2021-05-20T19:21:00.551646Z	error	cache	resource:default request:7aa8f9e4-627d-43f5-b28a-86b5801824d5 CSR hit non-retryable error (HTTP code: 0). Error: rpc error: code = Unauthenticated desc = request authenticate failure
2021-05-20T19:21:00.551683Z	error	cache	resource:default failed to rotate secret: rpc error: code = Unauthenticated desc = request authenticate failure
2021-05-20T19:26:00.409160Z	error	citadelclient	Failed to create certificate: rpc error: code = Unauthenticated desc = request authenticate failure
2021-05-20T19:26:00.409223Z	error	cache	resource:default request:153e49a7-c8e9-42a5-8957-d9db79ac4ac5 CSR hit non-retryable error (HTTP code: 0). Error: rpc error: code = Unauthenticated desc = request authenticate failure
Authenticator ClientCertAuthenticator at index 0 got error: no verified chain is found. Authenticator KubeJWTAuthenticator at index 1 got error: failed to validate the JWT: the service account authentication returns an error: [invalid bearer token, square/go-jose: error in cryptographic primitive]. Authenticator ClientCertAuthenticator at index 2 got error: no verified chain is found.
ROOTCA...
`)，

证书出问题了。。。

因为改了apiserver的参数，怀疑有遗留的脏数据，找了几天，才想起来是这个default-token

此时需要删除default-token,重启pod就可以了，
```bash
# kubectl get secret -n platform 
NAME                         TYPE                                  DATA   AGE
default-token-td4vp          kubernetes.io/service-account-token   3      12h
```
删除注入sidecar的ns中的default-token-xxx,会自动重建这个secret,如果不放心，可以把所有ns的这个secret都删了
然后，就可以愉快的重启(delete po 或者delete -f & apply -f)pod了，

唉，悲惨的适配