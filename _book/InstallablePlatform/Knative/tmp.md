本文基于 Knative [serving](https://github.com/knative/serving/tree/v0.17.0)，[eventing](https://github.com/knative/eventing/tree/v0.17.0) v0.17.0 版本

[官方文档](https://knative.dev/docs/)

# 安装

```
$ echo -e "$INTERNAL_INGRESS_HOST\thello-serving.default.example.com" | sudo tee -a /etc/hosts
curl http://hello-go.default.example.com
or
$ curl -H "Host: hello-serving.default.example.com" http://$INTERNAL_INGRESS_HOST
or
$ curl -H "Host: hello-serving.default.example.com" http://$INGRESS_HOST:$INGRESS_PORT
```

# 概念

## eventing



# 使用

## 基本功能-eventing

