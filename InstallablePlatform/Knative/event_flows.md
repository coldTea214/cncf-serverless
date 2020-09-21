Knative 提供了两种模型来定义事件流：Sequence 和 Parallel。前者是串型的，一个函数产出的 event 是下一个函数的输入；后者是并行的，多个函数可以收到同一个事件。下面会结合实例来说明

# Sequence

以下 demo 将3个 knative service 通过 event 串联起来

## 创建 knative service

先创建好3个应用

```
$ cat services.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: first
spec:
  template:
    spec:
      containers:
      - image: gcr.io/knative-releases/knative.dev/eventing-contrib/cmd/appender@sha256:e9ed1c369ba9a15675c12a58187b177089dfeeb5aecd406343dea7f1cbbcc6dd
        env:
        - name: MESSAGE
          value: " - Handled by 0"
---
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: second
spec:
  template:
    spec:
      containers:
      - image: gcr.io/knative-releases/knative.dev/eventing-contrib/cmd/appender@sha256:e9ed1c369ba9a15675c12a58187b177089dfeeb5aecd406343dea7f1cbbcc6dd
        env:
        - name: MESSAGE
          value: " - Handled by 1"
---
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: third
spec:
  template:
    spec:
      containers:
      - image: gcr.io/knative-releases/knative.dev/eventing-contrib/cmd/appender@sha256:e9ed1c369ba9a15675c12a58187b177089dfeeb5aecd406343dea7f1cbbcc6dd
        env:
        - name: MESSAGE
          value: " - Handled by 2"

$ kubectl apply -f services.yaml
```

appender 镜像的源码见：https://github.com/knative/eventing-contrib/tree/release-0.17/cmd/appender，基本功能就是收到 event 后，将环境变量 MESSAGE 内容追加到 event message 字段并返回 event

## 创建 sequence

```
$ cat sequence.yaml
apiVersion: flows.knative.dev/v1
kind: Sequence
metadata:
  name: sequence
spec:
  channelTemplate:
    apiVersion: messaging.knative.dev/v1
    kind: InMemoryChannel
  steps:
  - ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: first
  - ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: second
  - ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: third

$ kubectl apply -f sequence.yaml
```

上述命令对应的完整流程是：

* 创建了 sequence

	```
	$ kubectl get sequence
	NAME       URL                                                                  AGE     READY   REASON
	sequence   http://sequence-kn-sequence-0-kn-channel.default.svc.cluster.local   3h48m   True
	```
* eventing-controller 根据 sequence 描述创建了对应的 imc 及其 svc

	```
	$ kubectl get imc
	NAME                     URL                                                                  AGE     READY   REASON
	sequence-kn-sequence-0   http://sequence-kn-sequence-0-kn-channel.default.svc.cluster.local   3h48m   True
	sequence-kn-sequence-1   http://sequence-kn-sequence-1-kn-channel.default.svc.cluster.local   3h48m   True
	sequence-kn-sequence-2   http://sequence-kn-sequence-2-kn-channel.default.svc.cluster.local   3h48m   True
	$ kubectl get svc | grep sequence
	sequence-kn-sequence-0-kn-channel   ExternalName   <none>           imc-dispatcher.knative-eventing.svc.cluster.local      <none>                              3h48m
	sequence-kn-sequence-1-kn-channel   ExternalName   <none>           imc-dispatcher.knative-eventing.svc.cluster.local      <none>                              3h48m
	sequence-kn-sequence-2-kn-channel   ExternalName   <none>           imc-dispatcher.knative-eventing.svc.cluster.local      <none>                              3h48m
	```

## 验证

下面通过 pingsource 来触发完整 sequence

```
$ cat pingsource.yaml
apiVersion: sources.knative.dev/v1beta1
kind: PingSource
metadata:
  name: ping-source
spec:
  schedule: "* * * * *"
  jsonData: '{"message": "hello, world!"}'
  sink:
    ref:
      apiVersion: flows.knative.dev/v1
      kind: Sequence
      name: sequence

$ kubectl apply -f pingsource.yaml  
```

上述命令对应的完整流程是：

* 创建了 pingsource

	```
	$ kubectl get pingsource
	NAME          SINK                                                                 AGE     READY   REASON
	ping-source   http://sequence-kn-sequence-0-kn-channel.default.svc.cluster.local   3h50m   True
	```
* mtping 每分钟发送事件到 sink，也就是 sequence-kn-sequence-0-kn-channel
* 实际上就是发送到了 imc-dispatcher

	```
	$ kubectl get svc sequence-kn-sequence-0-kn-channel
	NAME                                TYPE           CLUSTER-IP   EXTERNAL-IP                                         PORT(S)   AGE
	sequence-kn-sequence-0-kn-channel   ExternalName   <none>       imc-dispatcher.knative-eventing.svc.cluster.local   <none>    3h54m
	```
* imc-dispatcher 查看对应的 subscriber，发送到 first pod

	```
	$ kubectl get imc sequence-kn-sequence-0 -o yaml | grep subscriberUri
	    subscriberUri: http://first.default.svc.cluster.local
	```
* first pod 收到 event 后，追加内容到 event message，再返回 event 到 imc-dispatcher
* imc-dispatcher 转发 event 到 sequence-kn-sequence-1-kn-channel

	```
	$ kubectl get imc sequence-kn-sequence-0 -o yaml | grep replyUri
	    replyUri: http://sequence-kn-sequence-1-kn-channel.default.svc.cluster.local
	```
* 接下来的流程基本是上述流程的重复，最终 event 会抵达 third pod

	```
	$ kubectl logs third-cd6sq-deployment-795b8c9f4c-ww7bl -c user-container | tail -1
	2020/09/14 14:50:00 [2020-09-14 14:50:00.000345656 +0000 UTC] /apis/v1/namespaces/default/pingsources/ping-source dev.knative.sources.ping: &{Sequence:0 Message:hello, world! - Handled by 0 - Handled by 1 - Handled by 2}
	```	

# Parallel

以下 demo 将同一个事件发送到两个处理函数

## 创建 knative service

先创建好4个应用

```
$ cat services.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: even-filter
spec:
  template:
    spec:
      containers:
      - image: villardl/filter-nodejs:0.1
        env:
        - name: FILTER
          value: |
            Math.round(Date.parse(event.time) / 60000) % 2 === 0
---
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: odd-filter
spec:
  template:
    spec:
      containers:
      - image: villardl/filter-nodejs:0.1
        env:
        - name: FILTER
          value: |
            Math.round(Date.parse(event.time) / 60000) % 2 !== 0
---
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: even-transformer
spec:
  template:
    spec:
      containers:
      - image: villardl/transformer-nodejs:0.1
        env:
        - name: TRANSFORMER
          value: |
            ({"message": "this is even!"})
---
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: odd-transformer
spec:
  template:
    spec:
      containers:
      - image: villardl/transformer-nodejs:0.1
        env:
        - name: TRANSFORMER
          value: |
            ({"message": "this is odd!"})

$ kubectl apply -f services.yaml
```

filter-nodejs 和 transformer-nodejs 源码没有放到 github 上，代码可以去镜像里确认，不过是 nodejs 的。这里说下大致的功能：

* filter 判断 FILTER 环境变量，如果为 true，则返回 event；如果为 false，则只返回 200 响应
* transformer 读取 TRANSFORMER 环境变量，并将其填充到 event message 发出

## 创建 parallel

```
$ cat parallel.yaml
apiVersion: flows.knative.dev/v1
kind: Parallel
metadata:
  name: odd-even-parallel
spec:
  channelTemplate:
    apiVersion: messaging.knative.dev/v1
    kind: InMemoryChannel
  branches:
  - filter:
      ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: even-filter
    subscriber:
      ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: even-transformer
  - filter:
      ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: odd-filter
    subscriber:
      ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: odd-transformer
  reply:
    ref:
      apiVersion: v1
      kind: Service
      name: event-display

$ kubectl apply -f parallel.yaml
```

这里复用一下 [event-display](https://coldtea214.gitbook.io/cncf-serverless/installableplatform/knative/event_source)

上述命令对应的完整流程是：

* 创建了 parallel

	```
	$ kubectl get parallel
	NAME                URL                                                                         AGE   READY   REASON
	odd-even-parallel   http://odd-even-parallel-kn-parallel-kn-channel.default.svc.cluster.local   59m   True
	```
* eventing-controller 依据 parallel 描述创建了对应的 imc 及其 svc

	```
	$ kubectl get imc
	NAME                              URL                                                                           AGE   READY   REASON
	odd-even-parallel-kn-parallel     http://odd-even-parallel-kn-parallel-kn-channel.default.svc.cluster.local     59m   True
	odd-even-parallel-kn-parallel-0   http://odd-even-parallel-kn-parallel-0-kn-channel.default.svc.cluster.local   59m   True
	odd-even-parallel-kn-parallel-1   http://odd-even-parallel-kn-parallel-1-kn-channel.default.svc.cluster.local   59m   True
	$ kubectl get svc | grep parallel
	odd-even-parallel-kn-parallel-0-kn-channel   ExternalName   <none>           imc-dispatcher.knative-eventing.svc.cluster.local      <none>                              59m
	odd-even-parallel-kn-parallel-1-kn-channel   ExternalName   <none>           imc-dispatcher.knative-eventing.svc.cluster.local      <none>                              59m
	odd-even-parallel-kn-parallel-kn-channel     ExternalName   <none>           imc-dispatcher.knative-eventing.svc.cluster.local      <none>                              59m
	```

## 验证

下面通过 pingsource 来触发完整 parallel

```
$ cat pingsource.yaml
apiVersion: sources.knative.dev/v1beta1
kind: PingSource
metadata:
  name: ping-source
spec:
  schedule: "* * * * *"
  jsonData: '{"message": "Even or odd?"}'
  sink:
    ref:
      apiVersion: flows.knative.dev/v1
      kind: Parallel
      name: odd-even-parallel

$ kubectl apply -f pingsource.yaml
```

上述命令对应的完整流程是：

* 创建了 pingsource

	```
	$ kubectl get pingsource
	NAME          SINK                                                                        AGE   READY   REASON
	ping-source   http://odd-even-parallel-kn-parallel-kn-channel.default.svc.cluster.local   5s    True
	```
* mtping 每分钟发送事件到 sink，也就是 odd-even-parallel-kn-parallel-kn-channel
* 实际上就是发送到了 imc-dispatcher

	```
	$ kubectl get svc odd-even-parallel-kn-parallel-kn-channel
	NAME                                       TYPE           CLUSTER-IP   EXTERNAL-IP                                         PORT(S)   AGE
	odd-even-parallel-kn-parallel-kn-channel   ExternalName   <none>       imc-dispatcher.knative-eventing.svc.cluster.local   <none>    75m
	```
* imc-dispatcher 查看对应的 subscriber，发送到 even-filter 和 odd-filter

	```
	$ kubectl get imc odd-even-parallel-kn-parallel -o yaml | grep subscriberUri
	    subscriberUri: http://even-filter.default.svc.cluster.local
	    subscriberUri: http://odd-filter.default.svc.cluster.local
	```
* 两个 filter 收到 event 后，根据 event time 判断是否返回 event
* imc-dispatcher 收到返回的 event 后，再根据 event 源查找对应的 replyUri 转发到下游 transformer 函数

	```
	$ kubectl get imc odd-even-parallel-kn-parallel -o yaml | grep replyUri
	    replyUri: http://odd-even-parallel-kn-parallel-0-kn-channel.default.svc.cluster.local
	    replyUri: http://odd-even-parallel-kn-parallel-1-kn-channel.default.svc.cluster.local
	```
* transformer 函数加上相应信息，发送 event 到 event-display

	```
	$ kubectl get imc odd-even-parallel-kn-parallel-0 -o yaml | grep replyUri
	    replyUri: http://event-display.default.svc.cluster.local/
	```
* 可以看到 event-display 间隔收到两个 transformer 发送的 event

	```
	$ kubectl logs -l app=event-display --tail=100 | grep -E "time|message"
	  time: 2020-09-15T11:32:00.000644492Z
	    "message": "this is even!"
	  time: 2020-09-15T11:33:00.000490241Z
	    "message": "this is odd!"
	```
