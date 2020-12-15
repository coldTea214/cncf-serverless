本文基于 [OpenFaas](https://github.com/openfaas/faas/tree/0.18.18) v0.18.18 版本和 [faas-netes](https://github.com/openfaas/faas-netes/tree/0.12.2) v0.12.2 版本

[官方文档](https://docs.openfaas.com/)

# 最近 release

开始学习之前，先看看本 repo 最近在干什么（截至 2020.12）:

## OpenFaas

* 0.20.0 [2020.11]
	* Add namespace in function name for metrics
* 0.19.0 [2020.10]
	* Security update for UI (AngularJS & Material)
* 0.18.18 [2020.07]
	* Remove service namespace from scale request
* 0.18.17 [2020.04]
	* Publish async requests to multiple topics
* 0.18.16 [2020.04]
	* Add QueueName to async requests

##  faas-netes

* 0.12.0 [2020.07]
	* Create new Policy client and type so that we can read Policy
configurations
	* Add linkerd auto injection
* 0.11.0 [2020.06]
	* Add cluster role and binding to operator
	* Expose nats monitoring metrics to prometheus
* 0.10.5 [2020.05]
	* Update apply REST handler

跟 fission、kubelsss 类似，没有太多功能性的改动。不过以 0.12.0（2020.07.15）、0.12.1（2020.07.20）、0.12.2（2020.07.20），发布密集、功能改动少，版本管理还是有点别具一格，所以后续只关注 0.12 -> 0.13 这种大版本变动，虽然看起来大版本间也没有太多变化

# 部署

结合实战来学习本 repo

## 安装

### 安装 client

```
$ curl -Lo faas-cli https://github.com/openfaas/faas-cli/releases/download/0.12.8/faas-cli \
	&& chmod +x faas-cli && sudo mv faas-cli /usr/local/bin/
```

### 安装 server

```
$ cd faas-netes # 进入 faas-netes 仓库
$ kubectl apply -f namespaces.yml
$ kubectl -n openfaas create secret generic basic-auth \
	--from-literal=basic-auth-user=admin \
	--from-literal=basic-auth-password="admin"
$ kubectl apply -f ./yaml/
```

确认部署组件都已正常工作

```
$ kubectl -n openfaas get pod
NAME                                 READY   STATUS    RESTARTS   AGE
alertmanager-7b84d7cf98-4clfx        1/1     Running   0          2m29s
basic-auth-plugin-6b69d6f8f7-hdg4s   1/1     Running   0          2m29s
faas-idler-69669f65b4-9dwgg          1/1     Running   0          2m29s
gateway-5bb669b8b9-7jnnr             2/2     Running   0          2m29s
nats-7b999bdb67-s9wsg                1/1     Running   0          2m29s
prometheus-88c57d976-fqmss           1/1     Running   0          2m28s
queue-worker-6c95cf8664-nm4sb        1/1     Running   0          2m28s
```

### 验证

```
$ export OPENFAAS_URL=http://127.0.0.1:31112
$ faas-cli login --password admin
$ faas-cli store deploy figlet
$ echo "hello, world!" | faas-cli invoke figlet
hello, world!
```

## 卸载

```
$ kubectl delete -f ./yaml/
$ kubectl delete -f namespaces.yml
```

# 架构

安装完 OpenFaas 之后，来梳理下安装的组件

| pod | 容器 | 命令 | 源码 repo |
|----|----|-----|--------|
| alertmanager | alertmanager | alertmanager --config.file ... | [alertmanager](https://github.com/prometheus/alertmanager) |
| basic-auth-plugin | basic-auth-plugin | handler | OpenFaas |
| faas-idler | faas-idler | faas-idler -dry-run=true | [faas-idler](https://github.com/openfaas-incubator/faas-idler/tree/0.3.0) |
| gateway | gateway | gateway | OpenFaas |
| | faas-netes | faas-netes | faas-netes |
| nats | nats | nats-streaming-server | [nats-streaming-server](https://github.com/nats-io/nats-streaming-server) |
| prometheus | prometheus | prometheus --config ... | [prometheus](https://github.com/prometheus/prometheus) |
| queue-worker | queue-worker | app | [nats-queue-worker](https://github.com/openfaas/nats-queue-worker) |

注：

* faas-idler 缺省安装情况下，是以 dry-run 运行的，也就是并没有在工作
* basic-auth 是个简单的鉴权实现，这里就不再详细说明了

整体架构如下：

![architecture](./architecture.png)

[图片来源](https://github.com/openfaas/faas-netes/tree/0.12.2)

其中：

* alertmanager、nats、prometheus 是社区其它项目组件
* queue-worker 是配合 nas 做异步函数 invoke 用的

而 k8s 体系下 faas 的相关实现，主要都是在 faas-netes 中实现的

# 概念

与 Fission 和 Kubeless 不同，OpenFaas 是从 docker 社区诞生的项目，所以最初的设计是面向过程式的，并没有引入 CRD

注：没有 CRD 当前是缺省模式，faas-netes 也追加了一种面向 CRD 的实现

# 使用

## 基本功能

本小节会按照“从零开始写一个 go 函数”的流程，来梳理功能

### 创建函数

```
$ faas-cli new hello-go --lang go
```

上述命令创建了一个 go 函数。实际完成的工作是：从[模板库](https://github.com/openfaas/templates.git)里获取模板到当前目录

```
$ tree -L 2
.
├── hello-go
│   └── handler.go
├── hello-go.yml
└── template
    ├── csharp
    ├── csharp-armhf
    ├── dockerfile
    ├── go
    ├── go-armhf
    ├── java11
    ├── java11-vert-x
    ├── java8
    ├── node
    ├── node12
    ├── node-arm64
    ├── node-armhf
    ├── php7
    ├── python
    ├── python3
    ├── python3-armhf
    ├── python3-debian
    ├── python-armhf
    └── ruby
```
	
### 填充 go 函数

单就我们本次流程而论，可以什么都不改。缺省模板代码：

```
$ cat hello-go/handler.go
package function

import (
	"fmt"
)

// Handle a serverless request
func Handle(req []byte) string {
	return fmt.Sprintf("Hello, Go. You said: %s", string(req))
}
```

### 编译 go 函数

```
$ faas-cli build -f hello-go.yml
```

上述命令本质上就是 docker build，最终生成一个含有 fwatchdog（一个 go web server） 和 go 代码编译包的工作镜像

### 推送 go 镜像

```
$ faas-cli push -f hello-go.yml
```

上述命令本质上就是 docker push

### 部署 go 函数

```
$ faas-cli deploy -f hello-go.yml
```

上述命令将刚刚编译、推送好的 docker 镜像部署到 k8s。涉及的完整流程是：

* faas-cli 发送 deploy 请求到 gateway
* gateway 将 deploy 请求转发给 faas-netes
* faas-netes 创建对应的 deployment、service

也只有到了这一步，faas-cli 才能看到函数（缺省模式下）。因为没有引入 CRD，faas-cli list 命令本质上也依赖从 deploy 获取信息，换句话说删除 deploy，list 返回也会变为空。而另外两个平台，则会根据 CR，重新创建 deploy

```
$ kubectl -n openfaas-fn get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
hello-go   1/1     1            1           11m
$ faas-cli list
Function                      	Invocations    	Replicas
hello-go                      	1              	1
```

### 触发 go 函数

```
$ echo -n "hello, world!" | faas-cli invoke hello-go
Hello, Go. You said: hello, world!
```

上述命令完成了 go 函数的触发。涉及的完整流程是：

* faas-cli 发送 invoke 命令到 gateway
* gateway 从 faas-netes 获取函数地址
* gateway 调用函数
* pod 里的 fwatchdog 进程收到请求，加载并运行用户函数

## 函数扩缩

OpenFaas 提供了多种扩缩实现：

* 基于 AlertManager 的实现
* 基于 HPA 的实现

除此之外，gateway 提供了 "scale up from zero" 能力，faas-idler 组件提供了 "scale down to zero" 的能力

OpenFaas 的扩缩配置比较分散，[参考文档](https://docs.openfaas.com/architecture/autoscaling/)

## 函数触发

上述小节以命令行的形式，手工触发函数调用，而在生产环境，OpenFaas 提供了以下几种触发方式：

* cron

	类似 Fission timer trigger，在 k8s 体系下通过 cronjob 实现，在其它平台提供了 cron connector
* http/webhooks

	缺省支持，通过以下地址触发

	```
	https://<gateway URL>:<port>/function/<function name>
	```
* async/nats streaming

	缺省支持，通过以下地址触发

	```
	https://<gateway URL>:<port>/async-function/<function name>
	```
* 自定义 trigger

	支持自定义 connector，监控自己关心的事件源，通过调用 gateway 来触发函数。OpenFaas 也累积了一些开发好的 connector，包括对接 AWS 等商业产品、redis 等开源产品的实现
