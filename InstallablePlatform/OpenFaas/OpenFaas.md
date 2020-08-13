本文基于 [OpenFaas](https://github.com/openfaas/faas/tree/0.18.18) v0.18.18 版本和 [faas-netes](https://github.com/openfaas/faas-netes/tree/0.12.2) v0.12.2 版本

[官方文档](https://docs.openfaas.com/)

# 最近 release

开始学习之前，先看看本 repo 最近在干什么

## OpenFaas

* 0.18.18 [2020.07]
	* Remove service namespace from scale request
* 0.18.17 [2020.04]
	* Publish async requests to multiple topics
* 0.18.16 [2020.04]
	* Add QueueName to async requests

##  faas-netes

* 0.12.0 [2020.07]
	* Create new Policy client and type so that we can read Policy
configurations
	* Add linkerd auto injection
* 0.11.0 [2020.06]
	* Add cluster role and binding to operator
	* Expose nats monitoring metrics to prometheus
* 0.10.5 [2020.05]
	* Update apply REST handler

跟 fission、kubelsss 类似，没有太多功能性的改动	