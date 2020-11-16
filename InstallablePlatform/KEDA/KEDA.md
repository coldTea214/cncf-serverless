本文基于 [KEDA](https://github.com/kedacore/keda/tree/v1.5.0) v1.5.0 版本

[官方文档](https://keda.sh/docs/1.5/)

# 最近 release

开始学习之前，先看看本 repo 最近在干什么（截至 2020.10）：

* 1.5.0 [2020.07]
	* Introduce Active MQ Artemis scaler
	* Introduce Redis Streams scaler
	* Introduce Cron scaler
* 1.4.0 [2020.04]
	* Fix scalers leaking
	* Extend RabbitMQ scaler to support count unacked messages
	* Handle nil pointers and empty arrays properly in Azure Monitor Scaler

KEDA 功能较为简单，目前的更新主要是丰富 scaler 和代码优化、修复

# 部署

结合实战来学习本 repo

## 安装

```
$ curl -Lo keda-1.5.0.tar.gz https://github.com/kedacore/keda/releases/download/v1.5.0/keda-1.5.0.tar.gz \
	&& tar -xf keda-1.5.0.tar.gz
$ kubectl apply -f keda-1.5.0/crds/ && kubectl apply -f keda-1.5.0
```

确认部署组件都已正常工作

```
$ kubectl -n keda get pod
NAME                                      READY   STATUS    RESTARTS   AGE
keda-metrics-apiserver-5f559d77d5-kbjdq   1/1     Running   0          6m22s
keda-operator-86dfcdf979-n568j            1/1     Running   0          6m22s
```

## 卸载

```
$ kubectl delete -f keda-1.5.0/crds/ && kubectl delete -f keda-1.5.0
```

# 架构

安装完 KEDA 之后，来梳理下安装的组件

| pod | 容器 | 命令 | 源码 repo |
|----|----|-----|--------|
| keda-metrics-apiserver | keda-metrics-apiserver | keda-adapter --secure-port=6443 ... | KEDA |
| keda-operator | keda-operator | keda --zap-level=info ... | KEDA |

operator 补足了 HPA 的能力，提供了缩容到0和从0扩容的能力；而 metrics-apiserver 则是作为 [kubernetes metrics server](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#support-for-custom-metrics)，配合 HPA 工作

![architecture](./architecture.png)

[图片来源](https://keda.sh/docs/1.5/concepts/#architecture)

上图中 KEDA 有3个逻辑组件：Metrics adapter（metrics-apiserver）、Controller（operator）和 Scaler。Scaler 代表 KEAD 能用于扩缩的事件源，分为内置和外置两部分

内置事件源包括但不限于：

* [Apache Kafka](https://keda.sh/docs/1.5/scalers/apache-kafka/)
* [AWS CloudWatch](https://keda.sh/docs/1.5/scalers/aws-cloudwatch/)
* [Azure Blob Storage](https://keda.sh/docs/1.5/scalers/azure-storage-blob/)
* [Cron](https://keda.sh/docs/1.5/scalers/cron/)
* [Huawei Cloudeye](https://keda.sh/docs/1.5/scalers/huawei-cloudeye/)
* ...

外部事件源需要按照一定的接口规范实现：[External Scalers](https://keda.sh/docs/1.5/concepts/external-scalers/)

# 概念

KEDA 引入了 scaledobject、scaledjob 和 triggerauthentication 三类 CRD

* scaledobject 记录了事件源和 Deployment、StatefulSet 或者任何实现了 /scale Custom Resource 的映射
* scaledjob 记录了事件源和 Job 的映射
* triggerauthentication 记录了访问事件源所需的鉴权数据
