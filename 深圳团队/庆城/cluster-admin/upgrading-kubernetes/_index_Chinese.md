---
标题：升级Kubernetes
---

> **先决条件：**以下选项仅对 [使用RKE启动的](/docs/cluster-provisioning/rke-clusters/)集群可用。

升级到最新版本的Rancher之后，您可以更新现有集群以使用最新支持的Kubernetes版本。

在新版本的Rancher发布之前，它会与Kubernetes的最新小版本进行测试，以确保兼容性。例如，Rancher v2.3.0使用Kubernetes v1.15.4、v1.14.7和v1.13.11进行了测试。有关在每个Rancher版本上测试的Kubernetes版本的详细信息，请参阅 [支持维护条款](https://rancher.com/support-maintenance-terms/all-supported-versions/rancher-v2.3.0/)。

从Rancher v2.3.0开始，添加了Kubernetes元数据功能，允许Rancher在不升级Rancher的情况下发布Kubernetes补丁版本。有关详细信息，请参阅 [Kubernetes metadata部分](/docs/admin-settings/k8s-metadata)

> **建议：** 在升级Kubernetes之前， [备份您的集群](/docs/backups).

1. 在 **Global** 视图中，找到要为其升级Kubernetes的集群。选择 **垂直省略号 (...) > 编辑**.

1. 展开 **集群选项**.

1. 从 **Kubernetes 版本** 下拉列表中，选择您希望用于集群的Kubernetes版本。

1. 点击 **保存**.

**结果：** Kubernetes开始为集群升级。在升级期间，您的集群将不可用。

