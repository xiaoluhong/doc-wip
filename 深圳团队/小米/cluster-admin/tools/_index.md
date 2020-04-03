---
title: 日志，监视和可视化的工具
---

Rancher包含Kubernetes中未包含的各种工具，可帮助您进行DevOps操作。 Rancher可以与外部服务集成，以帮助您的集群更高效地运行。工具分为以下几类：

<!-- TOC -->

- [通知和告警](#notifiers-and-alerts)
- [日志](#logging)
- [监控](#monitoring)
- [Istio](#istio)

<!-- /TOC -->

### 通知和告警

通知程序和告警是两个协同工作的功能，向您通知Rancher系统中的事件。

[通知](/docs/cluster-admin/tools/notifiers) 是通知您告警事件的服务。您可以配置通知接收者，以将告警通知发送给最适合采取措施的人员。通知可以通过Slack，电子邮件，PagerDuty，微信和Webhooks发送。

[告警](/docs/cluster-admin/tools/alerts) 是触发通知的规则。在接收告警之前，必须在Rancher中配置一个或多个通知接收者。告警范围可以在集群或项目级别设置。

### 日志

日志服务很有用，因为它使您能够：

- 捕获并分析集群的状态
- 在您的环境中发现趋势
- 将日志保存到集群之外的安全位置
- 随时了解容器崩溃，Pod驱逐或节点死亡等事件
- 更轻松地调试和排除故障

Rancher可以与Elasticsearch，Splunk，Kafka，Syslog和Fluentd集成。

有关详细信息，请参阅[日志部分](/docs/cluster-admin/tools/logging)

### 监控

_Available as of v2.2.0_

在Rancher，您可以通过领先的开源监控解决方案[Prometheus](https://prometheus.io/)监控集群节点，Kubernetes系统组件和软件部署的状态和过程。有关详细信息，请参阅[监控部分](/docs/cluster-admin/tools/monitoring)

### Istio

[Istio](https://istio.io/) 是一种开源工具，可使DevOps团队更轻松地观察，控制，故障排除和在复杂的微服务网络中的保证流量传输。有关如何在Rancher中启用Istio的详细信息，请参阅[Istio部分](/docs/cluster-admin/tools/istio)
