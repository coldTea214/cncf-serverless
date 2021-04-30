[官网地址](https://aws.amazon.com/lambda)

# 功能列表

AWS Lambda 功能[原文描述](https://aws.amazon.com/cn/lambda/features/)，原文里功能点描述的零散、无组织，这里对应归约到阿里云的功能组织样式：

* 丰富的触发器类型
	* AWS 各种产品触发器，如存储桶，对应原文描述里的“用自定义逻辑扩展其他 AWS 服务”
	* Http 触发器，对应原文“构建自定义后端服务”
* 多种编程语言支持，对应“自备代码”
	* Java
	* Go
	* PowerShell
	* Node.js
	* C#
	* Python
	* Ruby
	* Custom Runtime
* 丰富的资源类型
	* 按量实例，对应“自动扩展”
	* 预留实例，对应“精细的性能控制”
* 灵活的计量模式
	* 后付费模型，对应“只需按实际使用量付费”

除了能映射到阿里云功能的点之外，Lambda 也额外提及了一些功能：

* “完全自动化的管理”：即无需关心基础设施，就是 serverless 的基本优点之一
* “内置容错能力”：即跨可用区部署，高可用
* “连接到关系数据库”、“连接到共享文件系统”：即函数里可以使用 AWS 其它服务
* “运行代码以响应 Amazon CloudFront 请求”：即函数可全球部署，低延迟响应
* “编排多个函数”：即函数编排能力
* “集成化安全模型”：即利用密钥和网络隔离来保证函数安全
* “灵活的资源模型”：即可以限制函数的 CPU、带宽和磁盘 IO

# 产品更新

没找到 Lambda 相关的 release note，退而求其次，从开发人员指南的更新来反推 Lambda 功能改动。开发人员指南 release note：https://docs.aws.amazon.com/lambda/latest/dg/lambda-releases.html

截至 2021.04，相关的更新可以归纳为：

| 更新时间 | 更新内容 | 具体 |
|---------|--------|-----|
| 2021.03.02 | 功能优化 | The Lambda console has been redesigned to improve performance and consistency |
| 2021.01.27 | 编程语言支持 | Lambda now supports a new runtime for Node.js 14 |
| 2020.12.01 | 新增功能 | Lambda now supports functions defined as container images |
| 2020.11.05 | 集成 AWS 服务 | Lambda now supports Amazon MQ as an event source |
| 2020.10.08 | 函数集成外部服务 | Use Lambda extensions to augment your Lambda functions |
| 2020.08.11 | 集成 AWS 服务 | Lambda now supports Amazon MSK as an event source |

# 使用场景

| 场景 | 细分场景 | 案例描述 |
|-----|---------|--------|
| 数据处理 | 实时文件处理 | ![user-case1](./user-case1.png) |
| | 实时数据流处理 | ![user-case2](./user-case2.png) |
| | 机器学习 | 在 Aible，我们致力于以最低的运营成本提供最强大的 AI 技术。因此，我们使用 AWS Lambda 和 Serverless 进行机器学习训练和预测。借助 Serverless，我们可以更经济高效地运行各种机器学习工作负载，同时受益于快速迭代和扩展所需的突发计算资源，从而更好地开发 AI 技术以实现最佳业务影响 |
| 后端 | Web 应用程序 | ![user-case4](./user-case4.png) |
| | IoT 后端 | ![user-case5](./user-case5.png) |
| | 移动后端 | ![user-case6](./user-case6.png) |

# 重点功能

## 冷启动

官方文档：[AWS Lambda best practice](https://d1.awsstatic.com/whitepapers/serverless-architectures-with-aws-lambda.pdf)

上述文档是 Lambda 最佳实践合集，冷启动是其中的一部分。Lambda 针对冷启动的优化都是建议性质的，如尽量不要使用 VPC