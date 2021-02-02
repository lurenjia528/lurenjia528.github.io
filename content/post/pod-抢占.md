---
title: pod-抢占
#slug: chinese-test
date: 2020-05-21
categories:
- ubuntu
- k8s
- pod
tags:
- pod
- priorityClass
thumbnailImagePosition: left
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg
---
k8s中pod的资源抢占
<!--more-->

```bash
#cat low-priority.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 0
globalDefault: false
description: "This priority class should be used for Test pods only."
```
```bash
#cat high-priority.yaml 
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for Test pods only."
```
```bash
 cat busybox-low.yaml 
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: busybox2
  namespace: policy
spec:
  replicas: 4
  template:
    metadata:
      labels:
        app: busybox
        version: v1
        app1: demo-grpc
    spec:
      priorityClassName: low-priority 
      nodeSelector:
        node-role.kubernetes.io/node: "true"
      imagePullSecrets:
        - name: myregistrykey
      containers:
      - image: kylincloud2.hub/kube-system/busybox:0.0.1
        command: ["/bin/sh", "-c", "sleep 36000"]
        imagePullPolicy: IfNotPresent
        name: busybox
        resources:
          limits:
            cpu: 8000m
            memory: '4096M'
          requests:
            memory: '4096M'
            cpu: 8000m

```
```bash
 cat busybox-high.yaml 
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: busybox-high
  namespace: policy
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: busybox-high
        version: v1
        app1: demo-grpc
    spec:
      priorityClassName: high-priority 
      nodeSelector:
        node-role.kubernetes.io/node: "true"
      imagePullSecrets:
        - name: myregistrykey
      containers:
      - image: kylincloud2.hub/kube-system/busybox:0.0.1
        command: ["/bin/sh", "-c", "sleep 36000"]
        imagePullPolicy: IfNotPresent
        name: busybox-high
        resources:
          limits:
            cpu: 8000m
            memory: '4096M'
          requests:
            memory: '4096M'
            cpu: 8000m

```

```bash

root@master1:/nas/monitor-flink-druid-elk/pod-priority# kubectl get po -n policy -o wide 
NAME                        READY   STATUS    RESTARTS   AGE    IP              NODE     NOMINATED NODE   READINESS GATES
busybox2-86d849ddd9-bd6tq   0/1     Pending   0          110s   <none>          <none>   <none>           <none>
busybox2-86d849ddd9-djmtn   0/1     Pending   0          110s   <none>          <none>   <none>           <none>
busybox2-86d849ddd9-q52cb   1/1     Running   0          110s   10.244.247.10   node-2   <none>           <none>
busybox2-86d849ddd9-vddzc   0/1     Pending   0          110s   <none>          <none>   <none>           <none>


root@master1:/nas/monitor-flink-druid-elk/pod-priority# kubectl apply -f  busybox-high.yaml 
deployment.extensions/busybox-high created


root@master1:/nas/monitor-flink-druid-elk/pod-priority# kubectl get po -n policy -o wide 
NAME                            READY   STATUS    RESTARTS   AGE     IP             NODE     NOMINATED NODE   READINESS GATES
busybox-high-67d59d5477-nmg5p   1/1     Running   0          57s     10.244.247.6   node-2   <none>           <none>
busybox2-86d849ddd9-bd6tq       0/1     Pending   0          2m55s   <none>         <none>   <none>           <none>
busybox2-86d849ddd9-djmtn       0/1     Pending   0          2m55s   <none>         <none>   <none>           <none>
busybox2-86d849ddd9-dzd2h       0/1     Pending   0          57s     <none>         <none>   <none>           <none>
busybox2-86d849ddd9-vddzc       0/1     Pending   0          2m55s   <none>         <none>   <none>           <none>


root@master1:/nas/monitor-flink-druid-elk/pod-priority# kubectl get po -n policy -o wide 
NAME                            READY   STATUS    RESTARTS   AGE    IP             NODE     NOMINATED NODE   READINESS GATES
busybox-high-67d59d5477-nmg5p   1/1     Running   0          66s    10.244.247.6   node-2   <none>           <none>
busybox2-86d849ddd9-bd6tq       0/1     Pending   0          3m4s   <none>         <none>   <none>           <none>
busybox2-86d849ddd9-djmtn       0/1     Pending   0          3m4s   <none>         <none>   <none>           <none>
busybox2-86d849ddd9-dzd2h       0/1     Pending   0          66s    <none>         <none>   <none>           <none>
busybox2-86d849ddd9-vddzc       0/1     Pending   0          3m4s   <none>         <none>   <none>           <none>
```

不设置ResourceQuota的时候可以抢占
设置ResourceQuota后不抢占
```bash
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: policy
spec:
  hard:
#    pods: "3"
    requests.cpu: "10"
    requests.memory: 100Gi
    limits.cpu: "10"
    limits.memory: 200Gi
```
