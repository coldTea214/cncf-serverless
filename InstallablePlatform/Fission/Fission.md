本文基于 [Fission](https://github.com/fission/fission/tree/v1.10.0) v1.10.0 版本

[官方文档](https://docs.fission.io/docs/releases/1.10.0/)

# 最近 release

开始学习之前，先看看本 repo 最近在干什么

* 1.10.0 [2020.06]
	* S3 as a backend for Storage Service
	* Disabling env variable based discovery in Functions
	* Kube Context flag in Fission CLI
* 1.9.0 [2020.05]
	* Go 1.14 support
	* Function Level timeout
	* External Nats streaming
	* PodSecurityPolicy for Logger
* 1.8.0 [2020.02]
	* Go 1.13 support
	* Dry option to view the generated spec
	* Resource setting for fetcher

总的来说，基本是一些功能的补充和增强，没有大的更新

# 部署

结合实战来学习本 repo

## 安装

### 安装 client

```
$ curl -Lo fission https://github.com/fission/fission/releases/download/1.10.0/fission-cli-linux \
    && chmod +x fission && sudo mv fission /usr/local/bin/
```

### 安装 server

```
$ kubectl create ns fission
$ kubectl -n fission apply -f \
    https://github.com/fission/fission/releases/download/1.10.0/fission-core-1.10.0.yaml
```

如果目标 k8s 集群没有相应的 pv provision 机制，还需要：

* 修改 fission-storage-pvc storageClass 为 manual
* 再建个本地 pv 凑活一下

```
# fission-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fission-storage-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/fission-storage"
```

确认部署组件都已正常工作

```
$ kubectl -n fission get pod
NAME                           READY   STATUS    RESTARTS   AGE
buildermgr-956dc588-2475q      1/1     Running   0          6m32s
controller-c7b4bc759-5r8c9     1/1     Running   0          6m32s
executor-7489f6b8c-6vcfd       1/1     Running   0          6m32s
kubewatcher-77bd55f9dc-7p44k   1/1     Running   0          6m32s
router-7784db5696-lplqx        1/1     Running   0          6m32s
storagesvc-685c5dd95d-4csdh    1/1     Running   0          6m32s
timer-6565fbbccc-6btrg         1/1     Running   0          6m32s
```
	
### 验证

```
$ fission env create --name nodejs --image fission/node-env:1.10.0 --version 3 --poolsize 1
$ curl -LO https://raw.githubusercontent.com/fission/fission/master/examples/nodejs/hello.js
$ fission fn create --name hello-js --env nodejs --code hello.js
$ fission fn test --name hello-js
hello, world!
```

## 卸载

```
$ kubectl -n fission delete -f \
    https://github.com/fission/fission/releases/download/1.10.0/fission-core-1.10.0.yaml
$ kubectl delete ns fission
$ kubectl delete pv fission-storage-pv
```