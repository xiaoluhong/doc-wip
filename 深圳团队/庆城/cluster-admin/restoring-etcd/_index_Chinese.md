---
标题：恢复etcd
---

_从v2.2.0开始可用_

etcd备份和恢复由 [Rancher启用的Kubernetes集群](/docs/cluster-provisioning/rke-clusters/) 可以很容易地执行。etcd数据库的快照被获取并保存到本地的etcd节点或S3兼容的目标上。配置S3的优点是，如果所有etcd节点都丢失了，您的快照将被远程保存，并可用于恢复集群。

Rancher建议启用 [设置etcd重复快照的能力](/docs/cluster-admin/backing-up-etcd/#configuring-recurring-snapshots-for-the-cluster)，但是 [一次性快照](/docs/cluster-admin/backing-up-etcd/#one-time-snapshots) 也可以很容易地获取。Rancher允许从 [已保存的快照](#restoring-your-cluster-from-a-snapshot) 恢复，或者如果您没有任何快照，您仍然可以 [恢复etcd](#recovering-etcd-without-a-snapshot).

> **注意：** I如果您有任何Rancher启动了v2.2.0之前版本创建的Kubernetes集群，在升级Rancher之后，您必须 [编辑集群](/docs/cluster-admin/editing-clusters/) 并_保存_ 它， 以便启用 [已更新的快照功能](/docs/cluster-admin/backing-up-etcd/)。即使您已经使用v2.2.0之前的版本创建了快照，您也必须执行此步骤，因为旧的快照将无法用于通过UI备份和恢复etcd。

### 查看可用的快照

集群中所有可用快照的列表都是可用的。

1. 在 **全局** 视图中，导航到要查看快照的集群。

2. 从导航栏中点击 **工具>快照** 查看保存的快照列表。这些快照包括创建它们的时间戳。

### 从快照恢复集群

如果您的Kubernetes集群不可用了，您可以从快照中恢复集群。

1. 在 **全局** 视图中，导航到要查看快照的集群。

2. 点击 **垂直省略号 (...) > 恢复快照**。

3. 从可用快照的下拉菜单中选择要用于恢复集群的快照。点击 **保存**.

   > **注意：** 只有将集群配置为在S3上获取重复快照，才能从S3恢复快照。

**结果：** 集群将进入 `更新中` 状态，从快照恢复 `etcd` 节点的过程将启动。当集群返回到 `活动` 状态时，它将被恢复。

> **注意：** 如果您正在恢复一个具有不可用etcd节点的集群，建议在尝试恢复之前从Rancher中删除所有etcd节点。对于使用 [在基础设施提供商中托管的节点](/docs/cluster-provisioning/rke-clusters/node-pools/)创建的集群，将自动创建新的etcd节点。对于 [自定义集群](/docs/cluster-provisioning/rke-clusters/custom-nodes/)，请确保将新的etcd节点添加到集群中。

### 在没有快照的情况下恢复etcd

如果etcd节点组失去quorum, Kubernetes集群将报告失败，因为在Kubernetes集群中不能执行任何操作，例如部署工作负载。请查看Kubernetes集群中的 [etcd节点数量](/docs/cluster-provisioning/production/#count-of-etcd-nodes) 的最佳实践。如果您想恢复您的etcd节点集，请遵循以下说明:

1. 通过删除所有其他etcd节点，在集群中只保留一个etcd节点。

2. 在剩下的etcd节点上，运行以下命令:

   ```
   $ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock assaflavie/runlike etcd
   ```

   此命令输出etcd的运行命令，保存此命令供以后使用。

3. 停止上一步启动的etcd容器，将其重命名为 `etcd-old`。

   ```
   $ docker stop etcd
   $ docker rename etcd etcd-old
   ```

4. 使用步骤2中保存的命令并修改它:

   - 如果您最初拥有一个以上的etcd节点，那么您需要将 `--initial-cluster` 更改为只包含剩余的节点。
   - 在命令末尾添加`--force-new-cluster` 。

5. 运行修改后的命令。

6. 在单个节点启动并运行之后，Rancher建议向集群添加额外的etcd节点。如果您有一个 [自定义集群](/docs/cluster-provisioning/custom-clusters/) 并且希望重用旧节点，则需要在尝试将它们重新添加回集群之前 [清理节点](/docs/faq/cleaning-cluster-nodes/) 。

