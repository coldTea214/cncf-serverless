# serving

serving 引入了 Service(ksvc)、Route(rt)、Configuration(cfg) 和 Revision(rev) 4个面向用户的 CRD。4者的关系是：

![crd-serving](./crd-serving.png)

[图片来源](https://knative.dev/v0.14-docs/serving/#serving-resources)

具体来说：

* ksvc 包含了用户部署应用的所有信息
* rt 是应用版本路由信息，信息来自 ksvc
* cfg 包含应用容器模板信息，信息来自 ksvc
* rev 由 cfg 实例化而来，是 deploy 的信息来源。每修改一次 cfg，都会产生新的 rev

除了上面提及的 ksvc、rt、cfg 和 rev 外，实际上 serving 还有一些 CRD：Ingresses(king)、ServerlessService(sks)、PodAutoscaler(kpa)，这些会在使用小节结合实例说明

# eventing

eventing 引入了 Broker(broker)、Trigger(trigger) 2个面向用户的 CRD。具体来说：

* broker 是 event producer、consuer 之间的 "中间商"，基于 channel 来做转发，一个 broker 对应一个 channel provider
* trigger 绑定到 broker，记录了什么 consumer 订阅了什么样的事件

除了上面提及的 broker、trigger，实际上 eventing 还有一些 CRD：

* InMemoryChannel(imc)，是缺省 channel provider 关联的 CRD
* event source 相关 CRD，包括 ApiserverSource、ContainerSource、PingSource、SinkBinding
* event flows 相关 CRD，包括 Sequence、Parallel

这些会在使用小节结合实例说明
