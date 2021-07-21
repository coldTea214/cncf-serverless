开始学习之前，先看看本 repo 最近在干什么（截至 2021.06）

# serving

* [0.24.0](https://github.com/knative/serving/releases/tag/v0.24.0) [2021.06]
	* Allow dropping capabilities from a container's security context
	* Domainmapping can now specify a tls secret to be used as the https certificate
* [0.23.0](https://github.com/knative/serving/releases/tag/v0.23.0) [2021.05]
	* The stats scraping in the autoscaler is now sensitive to the EnableMeshPodAddressability setting
	* The state keeping in the activator is now sensitive to the EnableMeshPodAddressability setting
	* Tightens the heuristic for mesh being abled in the service scraper
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

# eventing

* [0.24.0](https://github.com/knative/eventing/releases/tag/v0.24.0) [2021.06]
	* no new feature
* [0.23.0](https://github.com/knative/eventing/releases/tag/v0.23.0) [2021.05]
	* no new feature
* [0.22.0](https://github.com/knative/eventing/releases/tag/v0.22.0) [2021.04]
	* no new feature
* [0.21.0](https://github.com/knative/eventing/releases/tag/v0.21.0) [2021.02]
	* Cloudevent traces available for PingSource
	* Message receiver supports customized liveness and readiness check
* [0.20.0](https://github.com/knative/eventing/releases/tag/v0.20.0) [2021.01]
	* 没啥 new feature

serving、eventing 整体的框架比较稳定，近期也没有太大变动
