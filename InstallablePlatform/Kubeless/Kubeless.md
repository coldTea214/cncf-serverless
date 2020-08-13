本文基于 [Kubeless](https://github.com/kubeless/kubeless/tree/v1.0.7) 1.0.7版本

[官方文档](https://kubeless.io/docs/quick-start/)

# 最近 release

开始学习之前，先看看本 repo 最近在干什么

* 1.0.7 [2020.06]
	* Feature: CronJob Triggers docs + CronJob create --payload arg
	* If HPA already exists, perform an update
	* Add soft pod anti affinity
* 1.0.6 [2020.01]
	* Perform a graceful shutdown for the function proxy on SIGINT & SIGTERM
	* Quote curl URL
	* Use new endpoint for the current deployment
* 1.0.5 [2019.10]
	* Use new apps/v1 endpoint
	* allows to provide default resources for initContainers

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