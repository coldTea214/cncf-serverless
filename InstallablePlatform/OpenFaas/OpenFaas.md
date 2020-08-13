本文基于 [OpenFaas](https://github.com/openfaas/faas/tree/0.18.18) v0.18.18 版本和 [faas-netes](https://github.com/openfaas/faas-netes/tree/0.12.2) v0.12.2 版本

[官方文档](https://docs.openfaas.com/)

# 最近 release

开始学习之前，先看看本 repo 最近在干什么

## OpenFaas

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

跟 fission、kubelsss 类似，没有太多功能性的改动

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
$ echo "hello world!" | faas-cli invoke figlet
hello world!
```

## 卸载

```
$ kubectl delete -f ./yaml/
$ kubectl delete -f namespaces.yml
```