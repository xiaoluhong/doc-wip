---
title: 将现有集群导入 Rancher
---

当管理一个导入的集群时, Rancher 连接到一个已经设置好的 Kubernetes 集群. 因此, Rancher 不提供 Kubernetes, 而只设置 Rancher 代理来与集群通信.

请记住, 编辑您的 Kubernetes 集群仍然需要在 Rancher 之外完成. 编辑集群的一些示例包括添加和删除节点、升级Kubernetes 版本以及更改 Kubernetes 组件参数.

#### 前提条件

如果您现有的 Kubernetes 集群已经定义了一个`集群管理员`角色, 那么您必须拥有这个`集群管理员`特权才能将集群导入Rancher.

为了获得特权, 您需要运行:

```plain
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin \
  --user [USER_ACCOUNT]
```

在运行`kubectl`命令以导入集群之前.

默认情况下，GKE 用户不会获得此权限，因此您需要在导入 GKE 集群之前运行该命令。要了解有关 GKE 基于角色的访问控制的详细信息，请单击 [此处](https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control).

#### 导入一个集群

1. 在 **集群** 页, 点击 **添加**.
2. 选择 **导入**.
3. 输入 **集群名称**.
4. {{< step_create-cluster_member-roles >}}
5. 单击 **创建**.
6. 这里显示了`集群管理员`特权的先决条件 (请参阅上面的**前提条件**), 其中包括了实现该先决条件的示例命令.
7. 将`kubectl`命令复制到剪贴板, 并根据 kubeconfig 的配置在指向要导入的集群的节点上运行它. 如果您不确定它是否正确配置, 在运行{{< product >}}中显示的命令之前, 运行`kubectl get nodes`来验证一下.
8. 如果您正在使用自签名证书, 您将收到`由未知机构签名的证书`消息. 要解决这个验证问题, 请把{{< product >}}中显示的`curl`开头的命令复制到剪贴板中. 然后根据 kubeconfig 的配置在指向要导入的集群的节点上运行它.
9. 在节点上运行完命令后, 单击 **完成**.
   {{< result_import-cluster >}}

> **注意:**
> 您不能重新导入当前在 Rancher 设置中处于活动状态的集群.
