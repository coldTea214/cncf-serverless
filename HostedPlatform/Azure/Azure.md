[官网地址](https://azure.microsoft.com/zh-cn/services/functions/)

# 功能列表

Azure Functions 功能（[原文描述](https://azure.microsoft.com/zh-cn/services/functions/#features)）：

* 改进端到端开发体验：Azure 提供了完整的函数开发（Visual Studio）、CI/CD（Azure pipeline）、监控（Azure Monitor）工具
* 简化复杂的业务流程挑战解决方案：引入了有状态函数的概念，详见[Durable Functions](https://docs.microsoft.com/zh-cn/azure/azure-functions/durable/durable-functions-overview?tabs=csharp)，目前仅支持 C#、JavaScript、Python、F# 和 PowerShell
* 在无需硬编码集成的情况下连接其他服务，以更快地开发解决方案：即 “Azure 服务事件触发” + “函数内使用 Azure 服务”
* 构建一次，随处部署：支持部署到 Azure、部署了 KEDA 的 kubernetes 集群等等
* 按你自己的方式进行开发：即支持多种编程语言

# 产品更新

Azure Functions release note：https://azure.microsoft.com/zh-cn/updates/?product=functions

截至 2020.12，相关的更新可以归纳为：

| 更新时间 | 更新内容 | 具体 |
|---------|--------|-----|
| 2020.12.09 | 开发者工具 | Java Azure Functions support on Linux is now generally available |
| 2020.11.18 | 编程语言支持 | Node.js 14 for Azure Functions is now available in public preview |
| 2020.10.28 | 开发者工具 | Generate a new function app from an OpenAPI specification |
| 2020.10.26 | 编程语言支持 | Java 11 support for Azure Functions is now generally available |
| 2020.09.22 | 编程语言支持 | Blazor and C# APIs now supported in Azure Static Web Apps |
| 2020.09.22 | 安全相关更新 | Azure Communication Services now available in public preview |
| 2020.09.02 | Durable Function 更新 | Durable Functions v2.3 现已发布 |
| 2020.08.19 | 编程语言支持 | PowerShell support in Durable Functions is in public preview |
| 2020.07.21 | 编程语言支持 | Java 11 for Azure Functions 现已推出预览版 |
| 2020.07.21 | 安全相关更新 | OpenID Connect support for Azure App Service and Azure Functions (in preview) |

Azure 的改动也比较常规，除了对 Durable Function 的持续投入

# 使用场景

| 场景 | 案例描述 |
|-----|--------|
| 无服务器 API | 具有 Node.js 或 Microsoft .NET 的无服务器 API |
| Web 应用程序 | ![user-case2](./user-case2.png) |
| 微服务 | ![user-case3](./user-case3.png) |
| 机器学习 | 具有无服务器体系结构的机器学习工作流 |
| 数据处理 | ![user-case5](./user-case5.png) |
| 云自动化 | ![user-case6](./user-case6.png) ||

# 重点功能

## 冷启动

官方文档：[了解和解决冷启动](https://docs.microsoft.com/zh-cn/azure/architecture/serverless-quest/functions-app-operations#understand-and-address-cold-starts)

Azure Functions 对冷启动的优化主要是靠[托管计划](https://docs.microsoft.com/zh-cn/azure/azure-functions/functions-scale)，本质上与阿里云预留实例一样。其它的都是建议性质的，如在函数应用中尽量使用异步操作

# 特色功能

## durable functions

官方文档：[什么是 Durable Functions](https://docs.microsoft.com/zh-cn/azure/azure-functions/durable/durable-functions-overview?tabs=csharp)

上述文档对 durable function 的使用场景描述的比较清楚，这里稍作一些说明：

* durable function 对一些常见模式做了封装，用普通函数也能实现，只是用户代码复杂一些。如[扇出/扇入场景](https://docs.microsoft.com/zh-cn/azure/azure-functions/durable/durable-functions-overview?tabs=csharp#pattern-2-fan-outfan-in)，普通函数如果跟踪所有扇出函数的完成状况，需要编写额外的控制代码
* 用其它的工具也能实现 durable function 的部分功能，只是管理更复杂、不够灵活。如[监视场景](https://docs.microsoft.com/zh-cn/azure/azure-functions/durable/durable-functions-overview?tabs=csharp#pattern-4-monitor)，普通函数的话可能需要配合时间触发器，还得自己调整控制轮询间隔等等，有时还需要管理多个触发器
* 其它云平台未必没有对应的能力，只是没有提炼、总结出独立的产品。如[函数链场景](https://docs.microsoft.com/zh-cn/azure/azure-functions/durable/durable-functions-overview?tabs=csharp#pattern-1-function-chaining)，某个函数失败只会在当前进度重试，不会全量重试，其它云平台可能也有对应能力
