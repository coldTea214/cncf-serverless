本文基于 [Kubeless](https://github.com/kubeless/kubeless/tree/v1.0.7) 1.0.7版本

[官方文档](https://kubeless.io/docs/quick-start/)

# 最近 release

开始学习之前，先看看本 repo 最近在干什么

* 1.0.7 [2020.06]
	* Feature: CronJob Triggers docs + CronJob create --payload arg
	* If HPA already exists, perform an update
	* Add soft pod anti affinity
* 1.0.6 [2020.01]
	* Perform a graceful shutdown for the function proxy on SIGINT & SIGTERM
	* Quote curl URL
	* Use new endpoint for the current deployment
* 1.0.5 [2019.10]
	* Use new apps/v1 endpoint
	* allows to provide default resources for initContainers

整体改动趋向于维护，没有引入什么新功能	