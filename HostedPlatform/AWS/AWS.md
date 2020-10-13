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

实在是没找到 Lambda 相关的 release note，这里只能退而求其次，从开发人员指南的更新来反推 Lambda 功能改动。开发人员指南 release note：https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/lambda-releases.html

截至 2020.09，相关的更新可以归纳为：

| 更新时间 | 更新内容 | 具体 |
|---------|--------|-----|
| 2020.08.12 | 编程语言支持 | 在 AL2 上支持 Java 8 和 自定义运行时 |
| 2020.08.11 | ASW 服务事件源 | 可提供 Amazon Managed Streaming for Apache Kafka 的新事件源 |
| 2020.08.10 | 安全相关更新 | 用于 Amazon VPC 设置的 IAM 条件键 |
| 2020.07.07 | 函数集成 AWS 服务 | Kinesis HTTP/2 流使用者的并发设置 |
| 2020.06.18 | 函数集成 AWS 服务 | Kinesis HTTP/2 流使用者的批处理时段 |
| 2020.06.16 | 函数集成 AWS 服务 | 对 Amazon EFS 文件系统的支持 |
| 2020.06.01 | 新增 demo | Lambda 控制台中的 AWS CDK 示例应用程序 |
| 2020.03.31 | 编程语言支持 | 在 AWS Lambda 中支持 .NET Core 3.1.0 运行时 |

AWS 近期的改动总体比较常规，基本都是为了更好的支持开发者以及融入 AWS 生态相关
