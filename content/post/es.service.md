---
title: es.service
slug: chinese-test
date: 2020-05-20
categories:
- ubuntu
- systemd
tags:
- es
- service
thumbnailImagePosition: left
thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/chinese-test-post/vintage-140.jpg
---
es.service
<!--more-->

# es.service
```bash

[Unit]
Description=Elasticsearch
Documentation=https://www.elastic.co
Wants=network-online.target
After=network-online.target
[Service]
Type=simple
RuntimeDirectory=elasticsearch
PrivateTmp=true
Environment=ES_HOME=/opt/elasticsearch7/elasticsearch-7.0.0/
Environment=ES_PATH_CONF=/opt/elasticsearch7/elasticsearch-7.0.0/config
Environment=PID_DIR=/var/run/elasticsearch
#Environment=ES_SD_NOTIFY=true
#EnvironmentFile=-/opt/elasticsearch7/elasticsearch-7.0.0/bin/elasticsearch-env
WorkingDirectory=/opt/elasticsearch7/elasticsearch-7.0.0/
User=es
Group=es
ExecStart=/opt/elasticsearch7/elasticsearch-7.0.0/bin/elasticsearch -p ${PID_DIR}/elasticsearch.pid --quiet
# StandardOutput is configured to redirect to journalctl since
# some error messages may be logged in standard output before
# elasticsearch logging system is initialized. Elasticsearch
# stores its logs in /var/log/elasticsearch and does not use
# journalctl by default. If you also want to enable journalctl
# logging, you can simply remove the "quiet" option from ExecStart.
StandardOutput=journal
StandardError=inherit
# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65535
# Specifies the maximum number of processes
LimitNPROC=4096
# Specifies the maximum size of virtual memory
LimitAS=infinity
# Specifies the maximum file size
LimitFSIZE=infinity
# Disable timeout logic and wait until process is stopped
TimeoutStopSec=0
# SIGTERM signal is used to stop the Java process
KillSignal=SIGTERM
# Send the signal only to the JVM rather than its control group
KillMode=process
# Java process is never killed
SendSIGKILL=no
# When a JVM receives a SIGTERM signal it exits with code 143
SuccessExitStatus=143
[Install]
WantedBy=multi-user.target
```
