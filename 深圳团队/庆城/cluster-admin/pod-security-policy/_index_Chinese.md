---
标题：添加Pod安全策略
---

> **先决条件:** 以下选项仅对 [使用RKE启动的](/docs/cluster-provisioning/rke-clusters/) 集群可用。

当您的集群运行具有安全敏感配置的pods时，为它分配一个 [pod安全策略](/docs/admin-settings/pod-security-policies/)，这是一组用于监视pods中的条件和设置的规则。如果pods不符合策略中指定的规则，策略将不允许它继续运行。

在创建集群时，可以分配pod安全策略。但是，如果您稍后需要放松或限制pod的安全性，您可以在编辑集群时更新策略。

1. 在 **全局** 视图中，找到要应用pod安全策略的集群。选择 **垂直省略号 (...) > 编辑**。

2. 展开 **集群选项**.

3. 在 **Pod安全策略支持**，选择启用。

   > **注意:** 此选项仅适用于 [有RKE创建](/docs/cluster-provisioning/rke-clusters/)的集群。

4. 从 **缺省Pod安全策略** 下拉列表中，选择要应用到集群的策略。

   Rancher提供了`限制` 和 `不限制`的 [策略](/docs/admin-settings/pod-security-policies/#default-pod-security-policies)  , 当然您也可以 [创建自定义策略](/docs/admin-settings/pod-security-policies/#default-pod-security-policies)。

5. 点击 **保存**.

**结果：** pod安全策略应用于集群和集群中的任何项目。

> **注意：** 在分配pod安全策略之前就已经运行的工作负载是不受限制的。即使它们不符合您的pod安全策略，在分配策略之前运行的工作负载也会继续运行。
>
> 要检查正在运行的工作负载是否通过您的pod安全策略，请克隆或升级它。

