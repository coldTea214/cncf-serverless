基本使用章节，都是手工构造、发送 event 来访问应用，在生产环境，一般通过 event source 来触发

下面会结合实战来介绍下缺省的4种 event source，在此之前，先创建一个 pod 作为后续所有生成 event 的 consumer

```
$ cat event-display.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: event-display
spec:
  replicas: 1
  selector:
    matchLabels: &labels
      app: event-display
  template:
    metadata:
      labels: *labels
    spec:
      containers:
      - name: event-display
        image: gcr.io/knative-releases/knative.dev/eventing-contrib/cmd/event_display@sha256:49dac8ea142b5d00bbb4b12f6a9cacb2b7826f36037e2d34d304cdcd289233c3
---
kind: Service
apiVersion: v1
metadata:
  name: event-display
spec:
  selector:
    app: event-display
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080

$ kubectl apply -f event-display.yaml
```

# apiserver source

apiserver source 监测 k8s 体系里的事件，生成对应的 cloudevent 事件

## 创建 rbac 规则

因为后续组件需要访问 k8s 事件，需要先创建相应的服务账户，绑定 event 的读能力

```
$ cat rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: events-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: event-watcher
rules:
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: k8s-ra-event-watcher
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: event-watcher
subjects:
- kind: ServiceAccount
  name: events-sa
  name: default
  
$ kubectl apply -f rbac.yaml
```

## 创建 apiserver source

使用上面的服务账户，创建 apiserver source

```
$ cat apiserversource.yaml
apiVersion: sources.knative.dev/v1beta1
kind: ApiServerSource
metadata:
  name: test-apiserver-source
spec:
  serviceAccountName: events-sa
  mode: Resource
  resources:
  - apiVersion: v1
    kind: Event
  sink:
    ref:
      apiVersion: v1
      kind: Service
      name: event-display

$ kubectl apply -f apiserversource.yaml
```

这里有一个 sink 字段，它定义了 event 的消费者，可以是以下几种对象：

* k8s service
* knative service
* knative channel
* knative broker

上述命令对应的完整流程是：

* 创建了 apiserversource

	```
	$ kubectl get apiserversource
	NAME                    SINK                                              AGE   READY   REASON
	test-apiserver-source   http://event-display.default.svc.cluster.local/   4s    True
	```
* eventing-controller 依据 apiserversource 创建了一个 pod，该 pod 执行的二进制是 apiserver\_receive\_adapter，该二进制基本功能是监测 k8s 体系下的事件，并转成 cloudevent 事件

	```
	$ kubectl get pod
	NAME                                                              READY   STATUS    RESTARTS   AGE
	apiserversource-test-apiserver-09b390f97a3b1206d5d4376634dv4b7q   1/1     Running   0          3m1s
	```

## 验证

```
$ kubectl run busybox --image=radial/busyboxplus:curl --restart=Never -- ls
$ kubectl delete pod busybox
$ kubectl logs -l app=event-display --tail=100 | grep message
    "message": "Created container busybox",
    "message": "Started container busybox",
```

可以看到 consumer 收到了 busybox 的创建、启动事件

# container source

container source 是容器内发送 cloudevent 事件，需要用户自己实现

## 创建 container source

```
$ cat containersource.yaml
apiVersion: sources.knative.dev/v1beta1
kind: ContainerSource
metadata:
  name: test-container-source
spec:
  template:
    spec:
      containers:
      - image: docker.io/coldtea214/heartbeats:1.0
        name: heartbeats
        args:
        - --period=1
        env:
        - name: POD_NAME
          value: "mypod"
        - name: POD_NAMESPACE
          value: "event-test"
  sink:
    ref:
      apiVersion: v1
      kind: Service
      name: event-display

$ kubectl apply -f containersource.yaml  
```

示例中 heartbeat 镜像对应源码：https://github.com/knative/eventing-contrib/blob/release-0.17/cmd/heartbeats/main.go ，配合输入参数，功能就是每秒发送一个 cloudevent 事件

上述命令对应的完整流程是：

* 创建了 container source

	```
	$ kubectl get containersource
	NAME                    SINK                                              AGE   READY   REASON
	test-container-source   http://event-display.default.svc.cluster.local/   26m   True
	```
* eventing-controller 依据 container source 指定的镜像创建了 pod

	```
	$ kubectl get pod
	NAME                                               READY   STATUS    RESTARTS   AGE
	test-container-source-deployment-9fff45549-z477s   1/1     Running   0          26m
	```
	其中，还注入了以下环境变量，给出了 event 的接收地址

	```
	$ kubectl get pod test-container-source-deployment-9fff45549-z477s -o yaml | grep -A1 "K_SINK"
	        - name: K_SINK
	          value: http://event-display.default.svc.cluster.local/
	```

## 验证

```
$ kubectl logs -l app=event-display --tail=100 | grep -E "time|id\""
  time: 2020-09-09T06:31:17.811801518Z
    "id": 655,
  time: 2020-09-09T06:31:18.811719004Z
    "id": 656,
  time: 2020-09-09T06:31:19.811791194Z
    "id": 657,
```

可以看到 consumer 收到了每分钟一次的 event

# ping source

ping source 本质上就是更早版本里的 cronjob source，可以周期性的生成 cloudevent 事件

## 创建 ping source

```
$ cat pingsource.yaml
apiVersion: sources.knative.dev/v1beta1
kind: PingSource
metadata:
  name: test-ping-source
spec:
  schedule: "*/1 * * * *"
  jsonData: '{"message": "hello, world!"}'
  sink:
    ref:
      apiVersion: v1
      kind: Service
      name: event-display

$ kubectl apply -f pingsource.yaml
```

上述命令对应的完整流程是：

* 创建了 ping source

	```
	$ kubectl get pingsource
	NAME               SINK                                              AGE    READY   REASON
	test-ping-source   http://event-display.default.svc.cluster.local/   106s   True
	```
* eventing controller 依据 pingsource 在 knative-eventing ns 下面创建了一个 pod，该 pod 执行的二进制是 mtping，该二进制的基本功能是读取 pingsource 中的时间安排 schedule 和消息 jsonData，按照时间发送包含该内容的 cloudevent 事件

	```
	$ kubectl -n knative-eventing get pod
	NAME                                    READY   STATUS    RESTARTS   AGE
	pingsource-mt-adapter-84b48b94b-pnlsc   1/1     Running   0          113s
	```

## 验证

```
$ kubectl logs -l app=event-display --tail=100 | grep -E "time|message"
  time: 2020-09-14T05:01:00.000313058Z
    "message": "hello, world!"
  time: 2020-09-14T05:02:00.000373127Z
    "message": "hello, world!"
```

可以看到 consumer 收到了每分钟一次，包含指定内容的事件

# sink binding

sink binding 可以用于将事件 producer 和 consumer 绑定起来

## 创建 producer

```
$ cat cronjob.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: heartbeat-cron
spec:
  schedule: "* * * * *"
  jobTemplate:
    metadata:
      labels:
        app: heartbeat-cron
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: single-heartbeat
            image: docker.io/coldtea214/heartbeats:1.0
            args:
            - --period=1
            env:
            - name: ONE_SHOT
              value: "true"
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            
$ kubectl apply -f cronjob.yaml
```

示例中 heartbeat 镜像对应源码：https://github.com/knative/eventing-contrib/blob/release-0.17/cmd/heartbeats/main.go ，配合输入参数，功能就是发送一个 cloudevent 事件后退出

## 创建 sink binding

与 containersource 场景不同，前者是创建时已经知道了 consumer，而后者则可以不知道。实际上如果仅创建上述 cronjob，则后续触发的 pod 都会失败，因为没有指定 consumer。这里通过创建 sinkbinding，将它绑定到相应的 consumer

```
$ cat sinkbinding.yaml
apiVersion: sources.knative.dev/v1beta1
kind: SinkBinding
metadata:
  name: test-sink-binding
spec:
  subject:
    apiVersion: batch/v1
    kind: Job
    selector:
      matchLabels:
        app: heartbeat-cron
  sink:
    ref:
      apiVersion: v1
      kind: Service
      name: event-display
  ceOverrides:
    extensions:
      hello: world

$ kubectl apply -f sinkbinding.yaml   
```

上述命令对应的完整流程是：

* 创建了 sinkbinding

	```
	$ kubectl get sinkbinding
	NAME                SINK                                              AGE   READY   REASON
	test-sink-binding   http://event-display.default.svc.cluster.local/   11m   True
	```
* eventing-webhook 后续在新建 pod 的时候，会根据此 sinkbinding 内容，为相应的 pod 注入对应内容

	```
	$ kubectl get pod | grep heartbeat
	# sinkbinding 创建前的 pod
	heartbeat-cron-1600062360-w2rm6   0/1     Error       0          13m
	# sinkbinding 创建后的 pod
	heartbeat-cron-1600063020-kr6bz   0/1     Completed   0          12m36s
	$ kubectl get pod heartbeat-cron-1600062360-w2rm6 -o yaml | grep -A1 "K_"
	$ kubectl get pod heartbeat-cron-1600063140-kr6bz -o yaml | grep -A1 "K_"
	    - name: K_SINK
	      value: http://event-display.default.svc.cluster.local/
	    - name: K_CE_OVERRIDES
	      value: '{"extensions":{"hello":"world"}}'
	```
	containersource 实际上环境变量直接在生成 deploy 的时候已经注入了，而 sinkbinding 为了做到 producer、consumer 解耦，则延后到通过 webhook 在 pod 新建时才注入

## 验证

```
$ kubectl logs -l app=event-display --tail=100 | grep -E "time|id\""
    "id": 1,
  time: 2020-09-14T05:50:03.250248562Z
    "id": 1,
  time: 2020-09-14T05:51:03.642889932Z
```

consumer 收到了每分钟 id 都为1的 cloudevent 事件

# 小结

除了上述内置 source 外，官方还支持 AWS SQS、Apache Kafka、GitHub 等事件源，第三方还支持 AWS DynamoDB、FTP / SFTP、VMware 等事件源，[完整列表](https://knative.dev/docs/eventing/sources/#knative-sources)
