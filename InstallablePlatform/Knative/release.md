开始学习之前，先看看本 repo 最近在干什么（截至 2021.04）

# serving

* [0.22.0](https://github.com/knative/serving/releases/tag/v0.22.0) [2021.04]
	* Added an autoscaling annotation to choose a different aggregation algorithm for the autoscaling metrics
* [0.21.0](https://github.com/knative/serving/releases/tag/v0.21.0) [2021.02]
	* Introduces autocreateClusterDomainClaim in config-network config map
	* Allow setting ReadOnlyRootFilesystem on the container's SecurityContext
	* Gradual Rollout is possible to set on individual Revisions
* [0.20.0](https://github.com/knative/serving/releases/tag/v0.20.0) [2021.01]
	* A feature flag is now available to enable hostAliases for Knative Services
	* Add AutoTLS support for DomainMapping
	* Gradual traffic rollout of the configuration targets is now available
* [0.19.0](https://github.com/knative/serving/releases/tag/v0.19.0) [2020.11]
	* Adds a Scale Down Delay feature
	* Adding cluster-wide flag max-scale-limit
	* deployments run with a minimal set of kernel capabilities

# eventing

* [0.22.0](https://github.com/knative/eventing/releases/tag/v0.22.0) [2021.04]
	* 没啥 new feature
* [0.21.0](https://github.com/knative/eventing/releases/tag/v0.21.0) [2021.02]
	* Cloudevent traces available for PingSource
	* Message receiver supports customized liveness and readiness check
* [0.20.0](https://github.com/knative/eventing/releases/tag/v0.20.0) [2021.01]
	* 没啥 new feature
* [0.19.0](https://github.com/knative/eventing/releases/tag/v0.19.0) [2020.11]
	* 没啥 new feature，主要是 bug fix 和现有功能的 update
* [0.18.0](https://github.com/knative/eventing/releases/tag/v0.18.0) [2020.09]
	* Allow MTChannelBroker TTL to be configured via ENV variable
	* PingSource adapter now uses bucket-based leader election
	* PingSource adapter deployment can now be customized at installation time
* [0.17.0](https://github.com/knative/eventing/releases/tag/v0.17.0) [2020.08]

serving、eventing 整体的框架比较稳定，近期也没有太大变动
