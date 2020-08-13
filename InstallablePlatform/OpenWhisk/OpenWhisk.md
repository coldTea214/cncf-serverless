本文基于 [Apache OpenWhisk](https://github.com/apache/openwhisk/tree/71b7d564ff60bf6e89be5410ffcf59f785d17a4a) 71b7d564ff60bf6e89be5410ffcf59f785d17a4a 版本。因为对 scala 并不熟，这个项目就简单介绍一下

# 架构

![architecture](./architecture.png)

[图片来源](https://github.com/apache/openwhisk/blob/71b7d564ff60bf6e89be5410ffcf59f785d17a4a/docs/about.md#the-internal-flow-of-processing)

OpenWhisk 复用了社区的 Nginx, Kafka, Docker, CouchDB 组件，新增了 Controller 和 Invoker 组件

这里给出一个简单的 demo：

* 用户编写函数代码

	```
	function main() {
	    console.log('Hello World');
	    return { hello: 'world' };
	}
	```
* 创建 action

	```
	wsk action create myAction action.js
	```
* 触发 action

	```
	wsk action invoke myAction --result
	```
	
create 是一个简单的 crud，下面结合架构图，来说一下 invoke 的工作流：

* wsk action invoke 命令向 nginx 发起了一个 http 请求
* nginx 转发请求到 controller
* controller 完成 request 的认证，并从 couchDB subjects 数据库获取信息完成鉴权
* 鉴权通过后，controller 从 couchDB whisks 数据库获取 action 信息
* controller 包含 Load Balancer 模块，它检查系统中所有 executor（称为 Invoker）的健康状况，获悉全局可用 invoker，发送信息到 kafka
* invoker 获取 message，kafka 返回 activationID 给用户（异步）
* invoker 启动 docker 容器，注入 action 代码，执行完后，将结果存储到 couchDB activations 数据库
* invoker 销毁容器

参考文档：

* [System overview](https://github.com/apache/openwhisk/blob/71b7d564ff60bf6e89be5410ffcf59f785d17a4a/docs/about.md)

# 概念

上文中提到了 action 等一些概念，本节会系统性的介绍 OpenWhisk 中的主要概念

* Namespace：namespace 是 OpenWhisk 组织资源对象的方式，像 actions, triggers, 和 rules 都归属于某一个 namespace
* Package：和 namespace 类似，一个 namespace 可以包含多个 package。对 actions 等对象来说，可以归属 package 也可以不归属。package 的主要作用是共享和复用，例如 /whisk.system/github 就包含了对 github webhook 的支持。具体内容可参考 [Using and creating OpenWhisk packages](https://github.com/apache/openwhisk/blob/71b7d564ff60bf6e89be5410ffcf59f785d17a4a/docs/packages.md)
* Action：action 就是用户定义的函数。支持多种语言，支持同步、异步调用，还可以控制 action 调用链顺序。借助 wsk 命令行可以方便的编写、触发、更新函数。具体内容可参考 [OpenWhisk Actions](https://github.com/apache/openwhisk/blob/71b7d564ff60bf6e89be5410ffcf59f785d17a4a/docs/actions.md)
* Trigger & Rule：通过 wsk action invoke 可以触发 action，但一般在生产环境，需要通过 trigger 和 rule 来触发。其中，trigger 由 feed 触发，rule 则绑定 trigger 和 action。具体内容可参考 [Creating triggers and rules](https://github.com/apache/openwhisk/blob/71b7d564ff60bf6e89be5410ffcf59f785d17a4a/docs/triggers_rules.md)
* Feed：feed 是触发函数的事件源，OpenWhisk 支持三种 feed：Hooks, Polling 和 Connections。Hooks 指由一个 webhook 触发，如 github 提交代码；Polling 指靠 action 去主动轮询；Connections 则是指另起一个服务，它来关注事件源，在获取事件后，主动通知 OpenWhisk。具体内容可参考 [Implementing feeds](https://github.com/apache/openwhisk/blob/71b7d564ff60bf6e89be5410ffcf59f785d17a4a/docs/feeds.md)

这里再对易混淆的 feed 和 trigger 对比说明一下。feed 是具体的事件源，trigger 是虚拟的触发器。具体来说，“博物馆人来了” 是一个 trigger，可以触发 “开门”、“亮灯” 等多个 action，此时，门口的 “摄像头1”、“摄像头2” 以及 “门口的大爷” 都是不同的 feed，它们都可以触发 trigger。从实现上来说，trigger 及后面所有对象、逻辑都是 OpenWhisk 内部逻辑，而 feed 则多是外部实现，它们最终通过 post 请求来触发 trigger

# 其它

OpenWhisk 还支持用户自定义自己的 runtime，具体内容可参考 [Developing a new Runtime with the ActionLoop proxy](https://github.com/apache/openwhisk/blob/71b7d564ff60bf6e89be5410ffcf59f785d17a4a/docs/actions-actionloop.md)