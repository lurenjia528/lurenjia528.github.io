---
title: envoy测试
slug: chinese-test
date: 2019-12-27
categories:
- ubuntu
- envoy
tags:
- envoy
thumbnailImagePosition: left
thumbnailImage: /img/envoy.png
---
redis启动错误
<!--more-->

# envoy测试

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      protocol: TCP
      address: 0.0.0.0
      port_value: 9901
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        protocol: TCP
        address: 0.0.0.0
        port_value: 10000
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          stat_prefix: ingress_http
          generate_request_id: false
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                route:
                  host_rewrite: www.baidu.com
                  cluster: service_webshell
          http_filters:
          - name: envoy.router
            config:
              dynamic_stats: false
  clusters:
  - name: service_webshell
    connect_timeout: 0.25s
    lb_policy: ROUND_ROBIN
    type: static
    hosts:
      - socket_address:
          address: 192.168.40.160
          port_value: 12345
    circuit_breakers:
      thresholds:
      - priority: HIGH
        max_connections: 1000
        max_pending_requests: 1000
        max_requests: 1000
      - priority: DEFAULT
        max_connections: 100
        max_pending_requests: 100
        max_requests: 100
```

访问envoyip:10000将转发到192.168.40.160：12345端口
