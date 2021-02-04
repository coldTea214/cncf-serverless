Installable Platform 是一些开源的 serverless 平台 or 增强组件。包括：

* [Apache Camel K](https://camel.apache.org/)：[项目地址](https://github.com/apache/camel-k)，Apache Camel K 是基于 [Apache Camel](https://github.com/apache/camel) 构建，运行在 Kubernetes 上的 Serverless 架构的轻量级集成平台。Apache Camel 是一款开源集成框架，通过这一框架能够快速的集成各类数据格式不同的生产或消费数据的系统
* [Apache OpenWhisk](https://openwhisk.apache.org/)：[项目地址](https://github.com/apache/openwhisk)，Apache 主导的开源 serverless 平台
* [AppScale](https://www.appscale.com/)：[项目地址](https://www.appscale.com/)，这是一个很老的项目了，2011年4月启动的项目，最初目的是提供和 GAE 兼容的平台，后续开始转向对 AWS 的支持
* [Fission](https://fission.io/)：[项目地址](https://github.com/fission/fission)，Platform9 创立的开源 serverless 平台
* [Keda](https://keda.sh/)：[项目地址](https://github.com/kedacore/keda)，Kubernetes Event-driven Autoscaling。最初是 Microsoft 主导的，后续捐赠给了 CNCF 孵化。这个项目只有一个目的，可以理解为一个强化版本的 HPA
* [Knative](https://github.com/knative/docs/)：[项目地址](https://github.com/knative/serving)，Google 主导的开源项目，Google 商业函数计算产品 Cloud Function 即基于此开源项目，目前仍积极维护中
* [KNIX](https://knix.io/)：[项目地址](https://github.com/knix-microfunctions/knix)，最初由 Nokia 贝尔实验室开发，2020年4月开源，是一个自带 UI、开发者工具的 faas 平台，可基于 Knative 工作。与其它 faas 不同，本身并不包含函数扩缩等相关逻辑
* [Kubeless](https://kubeless.io/)：[项目地址](https://github.com/kubeless/kubeless)，Bitnami 创立的开源 serverless 平台
* [Kyma](https://kyma-project.io/)：[项目地址](https://github.com/kyma-project/kyma)，这是一个旨在云原生世界，连接和扩展企业应用的项目
* [Nuclio](https://nuclio.io/)：[项目地址](https://github.com/nuclio/nuclio)，这是一个针对数据科学优化的 serverless 平台，这从它的卖点，例如与 Jupyter 的集成，就可以看出项目的倾向性
* [OpenFaaS](https://www.openfaas.com/)：[项目地址](https://github.com/openfaas/faas)，一个爱好者创立的开源项目，最初是基于 swarm 的，后面增加了对 k8s 的支持
* [PipelineAI](https://pipeline.ai/)：[项目地址](https://github.com/pipelineai/pipeline)，从名字也可以看出，这是一个侧重于 AI 相关的框架
* [Riff](https://projectriff.io/)：[项目地址](https://github.com/projectriff/riff)，这是一个基于 knative 的 faas 项目，提供了 cli，用于安装 knative 和其上部署函数，生成函数 invoker 等等
* [Virtual Kubelet](https://github.com/virtual-kubelet)：[项目地址](https://github.com/virtual-kubelet/virtual-kubelet)，通过类 kubelet api，让人可以定制 node agent，屏蔽实际节点，将请求转发至 serverless 容器平台，如 ACI 和 Fargate

其中，通用的 serverless 平台包括 Apache OpenWhisk、Fission、Kubeless 和 OpenFaas，这里先对这几个项目以及 KEDA、Knative 简单对比一下（截止 2021.01）：

| 项目 | 创立时间 | Star 数 | 主要语言 | 最新 release |
|-----|---------|--------|---------|------------- |
| Apache OpenWhisk | 2016.02 | 5.1k | Scala | 1.0.0(2020.11) |
| Fission | 2016.08 | 5.9k | Go | 1.11.2(2020.10) |
| [Fission 子项目] fission-workflows | 2017.07 | 303 | Go | 0.6.0(2018.10) |
| Kubeless | 2016.11 | 6.3k | Go | 1.0.8(2021.01) |
| OpenFaas | 2016.12 | 19.2k | Go | 0.20.7(2021.01) |
| [OpenFaas 子项目] faas-netes | 2017.07 | 1.7k | Go | 0.12.15(2021.01) |
| [OpenFaas 子项目][deprecated] faas-idler | 2018.05 | 59 | Go | 0.4.0(2020.11) |
| KEDA | 2019.02 | 2.8k | Go | 2.1.0(2021.01) |
| Knative/serving | 2018.01 | 3.6k | Go | 0.20.0(2021.01) |
| Knative/eventing | 2018.03 | 955 | Go | 0.20.1(2021.01) |

下面，针对通用的 serverless 平台和 KEDA、Knative 项目，再展开说明

参考文献：

* [Kyma - 轻松扩展和构建Kubernetes](https://cloud.tencent.com/developer/article/1548611)
* [Serverless：能简化数据科学项目吗](https://www.shangyexinzhi.com/article/177835.html)
* [An intro to project riff](https://files.gotocon.com/uploads/slides/conference_12/647/original/Eric%20BOTTARD%20-%20An%20intro%20to%20project%20riff.pdf)
