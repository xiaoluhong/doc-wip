---
title: 在ARM64上运行 (实验性)
---

_自v2.2.0起可用_

> **重要：**
>
> 在ARM64平台上运行目前是一项实验功能，Rancher尚未正式支持该功能。因此，我们不建议在生产环境中使用基于ARM64的节点。

使用ARM64平台时，可以使用以下选项：

- 在基于ARM64的节点上运行Rancher
  - 仅[安装Docker](/docs/installation/other-installation-methods/single-node-docker)
- 创建自定义集群并添加基于ARM64的节点
  - Kubernetes集群版本必须为1.12或更高
  - CNI网络提供商必须为[Flannel](/docs/faq/networking/cni-providers/#flannel)
- 导入包含基于ARM64的节点的群集
  - Kubernetes集群版本必须为1.12或更高

请参阅[集群选项](/docs/cluster-provisioning/rke-clusters/options/) 如何配置集群选项。

以下功能未经测试：

- 监控、告警、通知、流水线和日志
- 从应用商店启动应用程序
