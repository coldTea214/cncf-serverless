[官网地址](https://www.alibabacloud.com/zh/products/function-compute)

# 功能列表

阿里云函数计算提供的功能有：

* 丰富的触发器类型
	* 阿里云各种产品触发器，如对象存储、日志服务、消息队列、CDN
	* Http 触发器
	* 时间触发器
* 多种编程语言支持
	* Node.js
	* Python
	* Java
	* PHP
	* C#
	* Custom Runtime
* 便捷的开发工具
	* [funcraft](https://github.com/alibaba/funcraft)：阿里自研的开发、构建、部署工具
	* VSCode 插件
	* [Tools](https://coldtea214.gitbook.io/cncf-serverless/tools) 章节提及的 Serverless Devs
	* [Framework](https://coldtea214.gitbook.io/cncf-serverless/framework) 章节提及的 Midway Serverless
* 丰富的资源类型
	* 预留实例：针对冷启动的绕过方案
	* 按量实例：根据请求数动态扩缩
* 多样的实例类型
	* 弹性实例：代码包上限为50 MB，函数执行时长上限为600s
	* 性能实例：大规格实例，包含多种实例规格，资源上限更高，适配场景更多
* 灵活的计量模式
	* 按量付费模型：按实际使用计算资源计费
	* 资源包模型：预付费（包年包月）模型

# 产品更新

更新内容反映的是产品迭代的方向

阿里云函数计算 release note：https://www.alibabacloud.com/help/zh/doc-detail/154438.htm?spm=a2c63.l28256.a3.1.6fb63b2alJ9wob

截至 2021.06，相关的更新可以归纳为：

| 更新时间 | 更新内容 | 具体 |
|-------|--------|-----|
| 2021.06 | 有状态相关 | 有状态异步调用，除函数执行实例以外的硬件故障外，都将不会导致函数执行的中断或重复执行函数 |
| 2021.05 | 功能优化 | 增加默认服务角色，方便授权 |
| 2021.01 | 新增功能 | 层管理，将函数依赖的公共库提炼到层，以减少部署、更新时的代码包体积 |
| 2020.12 | 新增功能 | CICD部署 |
| 2020.12 | 功能优化 | 计费粒度由原来的百毫秒更改为毫秒 |
| 2020.11 | 新增功能 | 调用分析，系统会收集函数每次执行的指标信息，包括内存使用情况、函数执行时间... |
| 2020.11 | 开发者工具 | 编程模式扩展，在已有的 HTTP 服务器的模型中增加了 PreFreeze 和 PreStop webhooks |
| 2020.10 | 开发者工具 | Serverless Devs Tools，旨在可以更简单、快速的进行项目开发、创建、测试及部署 |

# 使用场景

使用场景是公司内外对产品形态的预期

阿里云给出的使用场景如下：

| 场景 | 案例描述 |
|-----|--------|
| Web 应用 | ![user-case1](./user-case1.png) |
| 实时数据处理 | ![user-case2](./user-case2.png) |
| AI 推理 | ![user-case3](./user-case3.png) |
| 视频转码 | ![user-case4](./user-case4.png) |

# 重点功能

## 冷启动

官方文档：[函数计算冷启动优化最佳实践](https://www.alibabacloud.com/help/zh/doc-detail/140338.htm?spm=a2c63.l28256.a3.52.c3bb3b2a9CGB5F)

阿里云提供的冷启动优化主要是预留实例，其它的都是对开发者的一些建议，如使用定时触发器预热函数；使用 Initializer 函数入口，函数计算会异步调用初始化接口