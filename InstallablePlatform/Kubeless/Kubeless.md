本文主要基于 [Kubeless](https://github.com/kubeless/kubeless/tree/v1.0.7) v1.0.7版本

[官方文档](https://kubeless.io/docs/quick-start/)

# 最近 release

开始学习之前，先看看本 repo 最近在干什么（截至 2021.01）：

* 1.0.8 [2021.01]
	* Update trigger controllers
	* add AWS Kinesis Trigger
	* Add secrets to runtime configuration
* 1.0.7 [2020.06]
	* Feature: CronJob Triggers docs + CronJob create --payload arg
	* If HPA already exists, perform an update
	* Add soft pod anti affinity
* 1.0.6 [2020.01]
	* Perform a graceful shutdown for the function proxy on SIGINT & SIGTERM
	* Quote curl URL
	* Use new endpoint for the current deployment

整体改动趋向于维护，没有引入什么新功能

# 部署

结合实战来学习本 repo

## 安装

### 安装 client

```
$ curl -Lo kubeless.zip https://github.com/kubeless/kubeless/releases/download/v1.0.7/kubeless_linux-amd64.zip \
	&& unzip kubeless.zip && sudo mv bundles/kubeless_linux-amd64/kubeless /usr/local/bin/
```

### 安装 server

```
$ kubectl create ns kubeless
$ kubectl apply -f https://github.com/kubeless/kubeless/releases/download/v1.0.7/kubeless-v1.0.7.yaml 
```

确认部署组件都已正常工作

```
$ kubectl -n kubeless get pod
NAME                                          READY   STATUS    RESTARTS   AGE
kubeless-controller-manager-7c9b96cc7-v2bgr   3/3     Running   0          2m50s
```

### 验证

```
$ cat hello.py
def hello(event, context):
  print event
  return event['data']
$ kubeless function deploy hello-py --runtime python2.7 --from-file hello.py --handler hello.hello
$ kubeless function ls # 等待 status 为 '1/1 READY'
$ kubeless function call hello-py --data 'hello, world!'
hello, world!
```

## 卸载

```
$ kubectl delete -f https://github.com/kubeless/kubeless/releases/download/v1.0.7/kubeless-v1.0.7.yaml
$ kubectl delete ns kubeless
```

# 架构

安装完 Kubeless 之后，来梳理下安装的组件

| pod | 容器 | 命令 | 源码 repo |
|----|----|-----|--------|
| controller | kubeless-function-controller | kubeless-function-controller | Kubeless |
| | http-trigger-controller | http-controller | [http-trigger](https://github.com/kubeless/http-trigger/tree/v1.0.2) |
| | cronjob-trigger-controller | cronjob-controller | [cronjob-trigger](https://github.com/kubeless/cronjob-trigger/tree/v1.0.3) |

两个 trigger 基本上名字就能说明它们的功能，所以 Kubeless 基本上所有逻辑都是在 controller 里实现的，也就没什么架构好说了

# 概念

Kubeless 引入了 function、trigger 两类 CRD。其中：

* function 记录了函数运行环境、源码、接口相关的信息
* trigger 记录了函数触发相关的信息

在使用小节会结合实例来详细说明

# 使用

## 基本功能

本小节会按照“从零开始写一个 go 函数”的流程，来梳理功能

### 编写 go 函数

```
$ cat go.mod
module function

go 1.14
$ cat hello.go
package kubeless

import (
	"github.com/kubeless/kubeless/pkg/functions"
)

func Hello(event functions.Event, context functions.Context) (string, error) {
	return "hello world!", nil
}
```

在 Kubeless 框架下写 go 代码，有两个基本要求：

* 必须带一个 go.mod（理论上其实没这必要，但 Kubeless 部分代码写死了）
* 接口实现依赖 ```github.com/kubeless/kubeless/pkg/functions``` 的引入

### 部署函数

```
$ kubeless function deploy hello-go --runtime go1.14 --handler hello.Hello --from-file hello.go --dependencies go.mod
```

上述命令指定 runtime 部署了 go 函数。涉及的完整流程是：

* kubeless client 创建了 function CR

	```
	# 用全名是因为跟 Fission function 重名，同时安装时，让命令无二意性
	$ kubectl get function.kubeless.io
	NAME       AGE
	hello-go   2m4s
	```
	function CR 里存储了包括源码在内的所有命令输入信息
* controller watch 到 function CR 后，创建 configmap 和函数 pod

	```
	$ kubectl get cm
	NAME       DATA   AGE
	hello-go   3      2m45s
	$ kubectl get pod
	NAME                        READY   STATUS       RESTARTS   AGE
	hello-go-5f4f76bd79-jnqnc   1/1     Running      0          2m49s
	```
	configmap 从 function 获取，并存储了代码和依赖相关信息。pod 除了运行函数的 runtime container 外，还有两个 init container：prepare 和 compile。prepare 从 configmap 挂载卷里拷贝代码等，compile 负责编译
	
### 触发函数

```
$ kubeless function call hello-go
hello world!
```

上述命令调用了刚刚创建的函数。涉及的完整流程是：

* kubeless client 根据函数名查询同名 service，获取地址
* kubeless client 调用函数
	
## 函数扩缩

Kubeless 需要主动创建扩缩规则并关联函数

```
$ kubeless autoscale create hello-go --min 1 --max 3 --value 50
```

上述命令本质上就是创建了一个 HPA，Kubeless 完全依赖 HPA 的能力，因此无法配置 min 为0

## 函数触发

上述小节以命令行的形式，手工触发函数调用，而在生产环境，Kubeless 提供了以下几种触发方式：

* http trigger

	与 Fission http trigger 类似，不过因为 Kubeless 仅有 controller 组件，所以依赖部署 ingress 来提供 http path 挂载，这里就不再演示了

* cronjob trigger

	与 Fission timer trigger 类似

	```
	$ kubeless trigger cronjob create minute --function hello-go --schedule "*/1 * * * *"
	```
* pubsub trigger

	与 Fission message queue trigger 类似，不过仅支持 Kafka，这里不再演示


