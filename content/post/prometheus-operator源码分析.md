---
title: prometheus-operator源码分析
slug: chinese-test
date: 2020-10-20
categories:
- go
- prometheus-operator
tags:
- go
- prometheus-operator
thumbnailImagePosition: left
thumbnailImage: /img/prometheus.jpeg
coverImage: /img/prometheus.jpeg
---
prometheus-operator源码分析
<!--more-->

# prometheus-operator

此次使用master分支

## 目录结构

```bash
.
├── bundle.yaml
├── CHANGELOG.md                                # changelog文件
├── cmd                                         # 命令行(二进制)
│   ├── operator                                # operator
│   │   ├── main.go
│   │   └── main_test.go
│   ├── po-docgen
│   │   ├── api.go
│   │   ├── compatibility.go
│   │   └── main.go
│   ├── po-lint
│   │   └── main.go
│   ├── po-rule-migration
│   │   └── main.go
│   └── prometheus-config-reloader              # config-reload(二进制)
│       ├── Dockerfile
│       ├── main.go
│       └── main_test.go
├── code-of-conduct.md
├── contrib
│   └── kube-prometheus
│       └── README.md
├── CONTRIBUTING.md
├── DCO
├── Dockerfile                                  # Dockerfile
├── Documentation                               # 文档
│   ├── additional-scrape-config.md
│   ├── api.md
│   ├── compatibility.md
│   ├── custom-configuration.md
│   ├── custom-metrics-elements.png
│   ├── design.md
│   ├── exposing-metrics.md
│   ├── high-availability.md
│   ├── logos
│   │   ├── prometheus-operator-logo.png
│   │   └── prometheus-operator-logo.svg
│   ├── network-policies.md
│   ├── rbac-crd.md
│   ├── rbac.md
│   ├── thanos.md
│   ├── troubleshooting.md
│   └── user-guides
│       ├── alerting.md
│       ├── basic-auth.md
│       ├── exposing-prometheus-and-alertmanager.md
│       ├── getting-started.md
│       ├── images
│       │   └── architecture.png
│       ├── linting.md
│       ├── monitoring-kubernetes-ingress.md
│       ├── running-exporters.md
│       ├── storage.md
│       └── webhook.md
├── example                                                 # 示例yaml
│   ├── additional-scrape-configs                           # prometheus额外的配置
│   │   ├── additional-scrape-configs.yaml
│   │   ├── prometheus-additional.yaml
│   │   ├── prometheus-cluster-role-binding.yaml
│   │   ├── prometheus-cluster-role.yaml
│   │   ├── prometheus-service-account.yaml
│   │   └── prometheus.yaml
│   ├── alertmanger-webhook
│   │   ├── Dockerfile
│   │   ├── main
│   │   ├── main.go
│   │   └── Makefile
│   ├── mixin
│   │   └── alerts.yaml
│   ├── networkpolicies                                   #网络策略
│   │   ├── alertmanager.yaml
│   │   ├── grafana.yaml
│   │   ├── kube-state-metrics.yaml
│   │   ├── node-exporter.yaml
│   │   └── prometheus.yaml
│   ├── non-rbac                                        # 非rbac
│   │   ├── prometheus-operator.yaml
│   │   └── prometheus.yaml
│   ├── prometheus-operator-crd                         # operator-crd
│   │   ├── monitoring.coreos.com_alertmanagers.yaml
│   │   ├── monitoring.coreos.com_podmonitors.yaml
│   │   ├── monitoring.coreos.com_probes.yaml
│   │   ├── monitoring.coreos.com_prometheuses.yaml
│   │   ├── monitoring.coreos.com_prometheusrules.yaml
│   │   ├── monitoring.coreos.com_servicemonitors.yaml
│   │   └── monitoring.coreos.com_thanosrulers.yaml
│   ├── rbac                                                # rbac
│   │   ├── prometheus
│   │   │   ├── prometheus-cluster-role-binding.yaml
│   │   │   ├── prometheus-cluster-role.yaml
│   │   │   ├── prometheus-service-account.yaml
│   │   │   └── prometheus.yaml
│   │   ├── prometheus-operator
│   │   │   ├── prometheus-operator-cluster-role-binding.yaml
│   │   │   ├── prometheus-operator-cluster-role.yaml
│   │   │   ├── prometheus-operator-deployment.yaml
│   │   │   ├── prometheus-operator-service-account.yaml
│   │   │   ├── prometheus-operator-service-monitor.yaml
│   │   │   └── prometheus-operator-service.yaml
│   │   └── prometheus-operator-crd
│   │       └── prometheus-operator-crd-cluster-roles.yaml
│   ├── storage                                                 # 存储
│   │   ├── persisted-prometheus.yaml
│   │   └── storageclass.yaml
│   ├── thanos                                                  # thanos(prometheus集群方案)
│   │   ├── prometheus-role-binding.yaml
│   │   ├── prometheus-role.yaml
│   │   ├── prometheus-servicemonitor.yaml
│   │   ├── prometheus-service.yaml
│   │   ├── prometheus.yaml
│   │   ├── query-deployment.yaml
│   │   ├── query-service.yaml
│   │   ├── sidecar-service.yaml
│   │   ├── thanos-ruler-service.yaml
│   │   └── thanos-ruler.yaml
│   └── user-guides                                        # 使用向导
│       ├── alerting
│       │   ├── alertmanager-example-service.yaml
│       │   ├── alertmanager-example.yaml
│       │   ├── alertmanager.yaml
│       │   ├── prometheus-example-rules.yaml
│       │   └── prometheus-example.yaml
│       └── getting-started
│           ├── example-app-deployment.yaml
│           ├── example-app-pod-monitor.yaml
│           ├── example-app-service-monitor.yaml
│           ├── example-app-service.yaml
│           ├── prometheus-admin-api.yaml
│           ├── prometheus-pod-monitor.yaml
│           ├── prometheus-service-monitor.yaml
│           └── prometheus-service.yaml
├── go.mod                                                  # 包管理
├── go.sum
├── helm
│   └── README.md
├── jsonnet
│   ├── mixin
│   │   ├── alerts
│   │   │   └── alerts.libsonnet
│   │   ├── alerts.jsonnet
│   │   ├── config.libsonnet
│   │   └── mixin.libsonnet
│   └── prometheus-operator
│       ├── alertmanager-crd.libsonnet
│       ├── jsonnetfile.json
│       ├── podmonitor-crd.libsonnet
│       ├── probe-crd.libsonnet
│       ├── prometheus-crd.libsonnet
│       ├── prometheus-operator.libsonnet
│       ├── prometheusrule-crd.libsonnet
│       ├── servicemonitor-crd.libsonnet
│       └── thanosruler-crd.libsonnet
├── kustomization.yaml
├── LICENSE
├── MAINTAINERS.md
├── Makefile
├── NOTICE
├── pkg                                                 # 源码
│   ├── admission                                 # prometheusrule的验证
│   │   ├── admission.go
│   │   ├── admission_test.go
│   │   ├── admissiontypes.go
│   │   └── jsonpatch.go
│   ├── alertmanager                              # alertmanager
│   │   ├── collector.go
│   │   ├── operator.go
│   │   ├── operator_test.go
│   │   ├── statefulset.go
│   │   └── statefulset_test.go
│   ├── api                                       # api的注册
│   │   └── api.go
│   ├── apis                                      # api的注册
│   │   └── monitoring
│   │       ├── register.go
│   │       └── v1
│   │           ├── doc.go
│   │           ├── register.go
│   │           ├── thanos_types.go
│   │           ├── types.go
│   │           ├── types_test.go
│   │           └── zz_generated.deepcopy.go
│   ├── client                                  # 客户端
│   │   ├── informers
│   │   │   └── externalversions
│   │   │       ├── factory.go
│   │   │       ├── generic.go
│   │   │       ├── internalinterfaces
│   │   │       │   └── factory_interfaces.go
│   │   │       └── monitoring
│   │   │           ├── interface.go
│   │   │           └── v1
│   │   │               ├── alertmanager.go
│   │   │               ├── interface.go
│   │   │               ├── podmonitor.go
│   │   │               ├── probe.go
│   │   │               ├── prometheus.go
│   │   │               ├── prometheusrule.go
│   │   │               ├── servicemonitor.go
│   │   │               └── thanosruler.go
│   │   ├── listers
│   │   │   └── monitoring
│   │   │       └── v1
│   │   │           ├── alertmanager.go
│   │   │           ├── expansion_generated.go
│   │   │           ├── podmonitor.go
│   │   │           ├── probe.go
│   │   │           ├── prometheus.go
│   │   │           ├── prometheusrule.go
│   │   │           ├── servicemonitor.go
│   │   │           └── thanosruler.go
│   │   └── versioned
│   │       ├── clientset.go
│   │       ├── doc.go
│   │       ├── fake
│   │       │   ├── clientset_generated.go
│   │       │   ├── doc.go
│   │       │   └── register.go
│   │       ├── scheme
│   │       │   ├── doc.go
│   │       │   └── register.go
│   │       └── typed
│   │           └── monitoring
│   │               └── v1
│   │                   ├── alertmanager.go
│   │                   ├── doc.go
│   │                   ├── fake
│   │                   │   ├── doc.go
│   │                   │   ├── fake_alertmanager.go
│   │                   │   ├── fake_monitoring_client.go
│   │                   │   ├── fake_podmonitor.go
│   │                   │   ├── fake_probe.go
│   │                   │   ├── fake_prometheus.go
│   │                   │   ├── fake_prometheusrule.go
│   │                   │   ├── fake_servicemonitor.go
│   │                   │   └── fake_thanosruler.go
│   │                   ├── generated_expansion.go
│   │                   ├── monitoring_client.go
│   │                   ├── podmonitor.go
│   │                   ├── probe.go
│   │                   ├── prometheus.go
│   │                   ├── prometheusrule.go
│   │                   ├── servicemonitor.go
│   │                   └── thanosruler.go
│   ├── k8sutil                             # k8s工具
│   │   ├── k8sutil.go
│   │   ├── k8sutil_test.go
│   │   ├── merge.go
│   │   └── merge_test.go
│   ├── listwatch                           # list/watch
│   │   ├── listwatch.go
│   │   ├── listwatch_test.go
│   │   ├── namespace_denylist.go
│   │   └── namespace_denylist_test.go
│   ├── namespace-labeler
│   │   ├── labeler.go
│   │   └── labeler_test.go
│   ├── operator                            # operator
│   │   ├── defaults.go
│   │   ├── image.go
│   │   ├── image_test.go
│   │   ├── operator.go
│   │   ├── server.go
│   │   ├── server_test.go
│   │   └── types.go
│   ├── prometheus                          # prometheus
│   │   ├── collector.go
│   │   ├── operator.go
│   │   ├── operator_test.go
│   │   ├── promcfg.go
│   │   ├── promcfg_test.go
│   │   ├── rules.go
│   │   ├── rules_test.go
│   │   ├── statefulset.go
│   │   └── statefulset_test.go
│   ├── thanos                              # thanos
│   │   ├── collector.go
│   │   ├── operator.go
│   │   ├── operator_test.go
│   │   ├── rules.go
│   │   ├── statefulset.go
│   │   └── statefulset_test.go
│   └── version
│       └── version.go
├── README.md
├── RELEASE.md
├── scripts                                             # 脚本
│   ├── check_license.sh
│   ├── create-minikube.sh
│   ├── delete-minikube.sh
│   ├── generate
│   │   ├── build-non-rbac-prometheus-operator.sh
│   │   ├── build-rbac-prometheus-operator.sh
│   │   ├── jsonnetfile.json
│   │   ├── prometheus-operator-non-rbac.jsonnet
│   │   └── prometheus-operator-rbac.jsonnet
│   ├── generate-bundle.sh
│   ├── generate-crds.go
│   ├── go.mod
│   ├── go.sum
│   ├── minikube-rbac.yaml
│   ├── run-external.sh
│   ├── tooling
│   │   └── Dockerfile
│   ├── tools.go
│   ├── travis-e2e.sh
│   └── travis-push-docker-image.sh
├── test
│   ├── e2e
│   │   ├── alertmanager_instance_namespaces_test.go
│   │   ├── alertmanager_test.go
│   │   ├── denylist_test.go
│   │   ├── main_test.go
│   │   ├── prometheus_instance_namespaces_test.go
│   │   ├── prometheus_test.go
│   │   ├── README.md
│   │   ├── remote_write_certs
│   │   │   ├── bad_ca.crt
│   │   │   ├── bad_ca.key
│   │   │   ├── bad_client.crt
│   │   │   ├── bad_client.key
│   │   │   ├── ca.crt
│   │   │   ├── ca.key
│   │   │   ├── client.crt
│   │   │   └── client.key
│   │   └── thanosruler_test.go
│   ├── framework
│   │   ├── admission-webhooks.go
│   │   ├── alertmanager.go
│   │   ├── cluster_role_binding.go
│   │   ├── cluster_role.go
│   │   ├── config_map.go
│   │   ├── context.go
│   │   ├── crd.go
│   │   ├── deployment.go
│   │   ├── event.go
│   │   ├── framework.go
│   │   ├── helpers.go
│   │   ├── ingress.go
│   │   ├── namespace.go
│   │   ├── pod.go
│   │   ├── probe.go
│   │   ├── prometheus.go
│   │   ├── prometheus_rule.go
│   │   ├── replication-controller.go
│   │   ├── ressources
│   │   │   ├── alertmanager-main-secret.yaml
│   │   │   ├── basic-auth-app-deployment.yaml
│   │   │   ├── default-http-backend-service.yml
│   │   │   ├── default-http-backend.yml
│   │   │   ├── nxginx-ingress-controller.yml
│   │   │   ├── prometheus-operator-mutatingwebhook.yaml
│   │   │   ├── prometheus-operator-validatingwebhook.yaml
│   │   │   └── prometheus-role-binding.yml
│   │   ├── role-binding.go
│   │   ├── secret.go
│   │   ├── service_account.go 
│   │   ├── service.go
│   │   └── thanosruler.go
│   └── instrumented-sample-app
│       ├── Dockerfile
│       ├── main.go
│       ├── Makefile
│       └── VERSION
└── VERSION

```
## 源码分析
启动入口：

cmd/operator/main.go 

```go
os.Exit(Main())
```

prometheus注册：

```go
247:	r := prometheus.NewRegistry()
	    po, err := prometheuscontroller.New(cfg, log.With(logger, "component", "prometheusoperator"), r)
	    if err != nil {
	    	fmt.Fprint(os.Stderr, "instantiating prometheus controller failed: ", err)
	    	return 1
	    }
...
327:	wg.Go(func() error { return po.Run(ctx.Done()) })
	    wg.Go(func() error { return ao.Run(ctx.Done()) })
	    wg.Go(func() error { return to.Run(ctx.Done()) })

```

pkg/prometheus/operator.go +182

```go
181: // New creates a new controller.
func New(conf Config, logger log.Logger, r prometheus.Registerer) (*Operator, error) {
	
}

484: // Run the controller.
func (c *Operator) Run(stopc <-chan struct{}) error {
	
...
509:	go c.worker() //执行队列

	go c.promInf.Run(stopc)
	go c.smonInf.Run(stopc)
	go c.pmonInf.Run(stopc)
	go c.probeInf.Run(stopc)
	go c.ruleInf.Run(stopc)
	go c.cmapInf.Run(stopc)
	go c.secrInf.Run(stopc)
	go c.ssetInf.Run(stopc)
	go c.nsMonInf.Run(stopc)
	if c.nsPromInf != c.nsMonInf {
		go c.nsPromInf.Run(stopc)
	}
	if err := c.waitForCacheSync(stopc); err != nil {
		return err
	}
// 添加事件处理
526:	c.addHandlers()

}

// 事件处理
// addHandlers adds the eventhandlers to the informers.
func (c *Operator) addHandlers() {
	c.promInf.AddEventHandler(cache.ResourceEventHandlerFuncs{
		AddFunc:    c.handlePrometheusAdd,    // 创建
		DeleteFunc: c.handlePrometheusDelete, //删除
		UpdateFunc: c.handlePrometheusUpdate, //更新
	})
...
}

// worker runs a worker thread that just dequeues items, processes them, and
// marks them done. It enforces that the syncHandler is never invoked
// concurrently with the same key.
func (c *Operator) worker() {
	for c.processNextWorkItem() {
	}
}

func (c *Operator) processNextWorkItem() bool {
	key, quit := c.queue.Get()
	if quit {
		return false
	}
	defer c.queue.Done(key)

	c.metrics.ReconcileCounter().Inc()
	err := c.sync(key.(string))
	if err == nil {
		c.queue.Forget(key)
		return true
	}

	c.metrics.ReconcileErrorsCounter().Inc()
	utilruntime.HandleError(errors.Wrap(err, fmt.Sprintf("Sync %q failed", key)))
	c.queue.AddRateLimited(key)

	return true
}


func (c *Operator) sync(key string) error {
	...
1241:	sset, err := makeStatefulSet(*p, &c.config, ruleConfigMapNames, newSSetInputHash)
...
}
```

pkg/prometheus/statefulset.go 

```go
func makeStatefulSet(
	p monitoringv1.Prometheus,
	config *Config,
	ruleConfigMapNames []string,
	inputHash string,
) (*appsv1.StatefulSet, error) {
	// p is passed in by value, not by reference. But p contains references like
	// to annotation map, that do not get copied on function invocation. Ensure to
	// prevent side effects before editing p by creating a deep copy. For more
	// details see https://github.com/prometheus-operator/prometheus-operator/issues/1659.
	p = *p.DeepCopy()
...
}
```