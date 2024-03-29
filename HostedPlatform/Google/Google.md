[官网地址](https://cloud.google.com/functions/)

# 功能列表

Google cloud functions 功能（[原文描述](https://cloud.google.com/functions#section-14)）：

* 连接和扩展服务以构建复杂应用：即函数内使用 Google 服务
* 端到端开发和可诊断性：从代码到部署，都具备集成的监控功能。利用 Cloud Trace 和 Cloud Debugger 使您的应用具备全面的可监测性和可诊断性
* 自动扩缩：即基于负载的自动扩缩
* 无需管理服务器
* 运行代码以响应事件：即 Google 服务事件触发
* 用多少、付多少
* 采用开源技术，避免供应商锁定：基于 [Knative](https://coldtea214.gitbook.io/cncf-serverless/installableplatform/knative)

# 产品更新

Google cloud functions release note：https://cloud.google.com/functions/docs/release-notes

截至 2021.06，相关的更新可以归纳为：

| 更新时间 | 更新内容 | 具体 |
|---------|--------|-----|
| 2021.06.07 | 编程语言支持 | Cloud Functions now supports Ruby 2.6 and 2.7 at the General Availability release level |
| 2021.04.19 | 编程语言支持 | Cloud Functions has added support for a new runtime, PHP 7.4, in Preview. |
| 2021.01.11 | 编程语言支持 | Cloud Functions has added support for a new runtime, Ruby, in Preview |
| 2020.11.19 | 编程语言支持 | Cloud Functions has added support for a new runtime, .NET |
| 2020.11.11 | 编程语言支持 | Cloud Functions now supports Python 3.8 |
| 2020.10.13 | 开发者工具 | In runtimes that use buildpacks you can now configure aspects of your build |
| 2020.08.26 | 编程语言支持 | Node 10 runtime now builds container images in the user's project... |

# 使用场景

| 场景 | 案例描述 |
|----|--------|
| 与第三方服务及 API 集成 | ![user-case1](./user-case1.svg) |
| 无服务器移动后端 | ![user-case2](./user-case2.svg) |
| 无服务器 IoT 后端 | ![user-case3](./user-case3.svg) |
| 实时文件处理 | ![user-case4](./user-case4.svg) |
| 实时流处理 | ![user-case5](./user-case5.svg) |
| 虚拟助理和对话式体验 | ![user-case6](./user-case6.svg) |
| 视频和图片分析 | ![user-case7](./user-case7.svg) |
| 情感分析 | ![user-case8](./user-case8.svg) |

# 重点功能

## 冷启动

官方文档：[最佳实践-提示与技巧](https://cloud.google.com/functions/docs/bestpractices/tips)

这篇文档是全局的一些最佳实践，冷启动是其中的一部分。Cloud Functions 针对冷启动的优化都是些建议性质的
