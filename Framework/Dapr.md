# 概述

[dapr](https://github.com/dapr/dapr) 是微软19年6月主导开源的项目

简单来说，dapr 以进程或 sidecar 容器形式部署，通过 sdk 封装了 http、grpc 接口，对外提供了实现微服务所需的一系列功能（best practice building blocks），如事件驱动、状态管理等。dapr 项目由社区驱动、平台无关、语言无关，同时性能好、轻量，也能用于 iot 和 edge 场景

当前提供的功能有：

* 支持可选供应商的事件驱动 pub-sub 系统，提供“至少一次”语义
* 支持可选供应商的输入、输出绑定
* 支持可选 data stores 的状态管理
* 服务到服务的发现和调用
* 可选的有状态模型：Strong/Eventual consistency, First-write/Last-write wins
* 跨平台的 virtual actors
* 密钥管理
* 限流
* 可观测

# 参考文献

* [重磅！微软开源微服务构建软件 Dapr](https://www.infoq.cn/article/ygNxYaTIxdBjejcyjv8Y)
