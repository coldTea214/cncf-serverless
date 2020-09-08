开始学习之前，先看看本 repo 最近在干什么

# serving

* [0.17.0](https://github.com/knative/serving/releases/tag/v0.17.0) [2020.08]
	* Meta
		* initialScale annotation to control the initial deployment size
		* net-contour and net-kourier have moved to Beta
	* Autoscaling
		* Launched new KPA statuses, which permit significant simplification of the state machine in revision and KPA itself
		* Concurrency & stat reporting rewrite in Activator
	* Core API
		* Leader Election enabled by default
		* Adopt a two-lane work queue for our controllers to prevent starvation during global re-syncs pkg
	* Networking
		* Code in knative/serving/pkg/network is completely moved to knative/networking repo
		* Kingress (net-istio) introduces RewriteHost feature
* [0.16.0](https://github.com/knative/serving/releases/tag/v0.16.0) [2020.07]
* [0.15.0](https://github.com/knative/serving/releases/tag/v0.15.0) [2020.05]

# eventing

* [0.17.0](https://github.com/knative/eventing/releases/tag/v0.17.0) [2020.08]
	* ContainerSource、SinkBinding is now in v1beta1
	* Eventing conformance tests now can validate Sources status conformance
	* PingSource now supports setting the time zone
* [0.16.0](https://github.com/knative/eventing/releases/tag/v0.16.0) [2020.07]
* [0.15.0](https://github.com/knative/eventing/releases/tag/v0.15.0) [2020.05]

serving、eventing 实际更新的内容很多，项目在很活跃的发展，全量更新可以自行点击链接确认
