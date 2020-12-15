开始学习之前，先看看本 repo 最近在干什么（截至 2020.12）

# serving

* [0.19.0](https://github.com/knative/serving/releases/tag/v0.19.0) [2020.11]
	* Adds a Scale Down Delay feature
	* Adding cluster-wide flag max-scale-limit
	* deployments run with a minimal set of kernel capabilities
* [0.18.0](https://github.com/knative/serving/releases/tag/v0.18.0) [2020.09]
	* Meta
		* Serving v1alpha1 & v1beta1 will EOL in our next release (v0.19)
		* Kubernetes minimum version has changed to v1.17
	* Autoscaling
		* Improving performance, memory allocation, testability and readability of the codebase
		* Max Scale Limit configuration
	* Core API
		* Add support for serviceAccountToken in projected volumes
		* Added RuntimeClassName feature flag
	* Networking
		* Net-contour is moved to stable stage
		* Allow disabling AutoTLS with an annotation
* [0.17.0](https://github.com/knative/serving/releases/tag/v0.17.0) [2020.08]
* [0.16.0](https://github.com/knative/serving/releases/tag/v0.16.0) [2020.07]

# eventing

* [0.19.0](https://github.com/knative/eventing/releases/tag/v0.19.0)
	* 没啥 new feature，主要是 bug fix 和现有功能的 update
* [0.18.0](https://github.com/knative/eventing/releases/tag/v0.18.0) [2020.09]
	* Allow MTChannelBroker TTL to be configured via ENV variable
	* PingSource adapter now uses bucket-based leader election
	* PingSource adapter deployment can now be customized at installation time
* [0.17.0](https://github.com/knative/eventing/releases/tag/v0.17.0) [2020.08]
* [0.16.0](https://github.com/knative/eventing/releases/tag/v0.16.0) [2020.07]

serving、eventing 实际更新的内容比较多，全量更新可以自行点击链接确认
