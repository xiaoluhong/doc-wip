---
标题：节点和节点池
---

在Rancher中启动Kubernetes集群之后，可以从集群的**节点** 选项卡管理各个节点。根据提供集群的 [使用的选项](/docs/cluster-provisioning/#cluster-creation-in-rancher) ，可以使用不同的节点选项。

本页涵盖以下主题:

- [每种类型集群的节点选项](#node-options-for-each-type-of-cluster)
- [隔离和驱逐节点](#cordoning-and-draining-nodes)
- [编辑节点](#editing-a-node)
- [查看节点API](#viewing-a-node-api)
- [删除节点](#deleting-a-node)
- [扩展节点](#scaling-nodes)
- [SSH到由基础设施提供商托管的节点](#ssh-into-a-node-hosted-by-an-infrastructure-provider)
- [管理节点池](#managing-node-pools)

要管理单个节点，请浏览要管理的集群，然后从主菜单中选择 **节点** 。您可以通过单击节点的* **省略号** 图标 (**...**)来打开该节点的选项菜单。

> **注意：** 如果您想管理 _集群_ 而不是单个节点，请参阅 [编辑集群](/docs/k8s-in-rancher/editing-clusters).

## 每种类型集群的节点选项

下表列出了Rancher中每个 [类型的集群](/docs/cluster-provisioning/#cluster-creation-options) 可用的节点选项。单击 **选项** 列中的链接以获得关于每个功能模块的更详细信息。

| Option                                                       | [由基础设施提供商托管的节点][1] | [自定义的节点][2] | [托管的集群][3] | [导入的节点][4] | 描述                                              |
| ------------------------------------------------------------ | ------------------------------- | ----------------- | --------------- | --------------- | ------------------------------------------------- |
| [隔离](#cordoning-a-node)                                    | ✓                               | ✓                 | ✓               |                 | 将节点标记为不可调度。                            |
| [驱逐](#draining-a-node)                                     | ✓                               | ✓                 | ✓               |                 | 将节点标记为不可调度的， _并_ 将所有pod驱逐出去。 |
| [编辑](#editing-a-node)                                      | ✓                               | ✓                 | ✓               |                 | 输入节点的自定义名称、描述、标签或污点。          |
| [查看 API](#viewing-a-node-api)                              | ✓                               | ✓                 | ✓               |                 | 查看API数据。                                     |
| [删除](#deleting-a-node)                                     | ✓                               | ✓                 |                 |                 | 从集群中删除有缺陷的节点。                        |
| [下载Keys](#ssh-into-a-node-hosted-by-an-infrastructure-provider) | ✓                               |                   |                 |                 | 下载SSH密钥以便SSH到节点。                        |
| [节点扩展](#scaling-nodes)                                   | ✓                               |                   |                 |                 | 增加或减少节点池中的节点数。                      |

[1]: /docs/cluster-provisioning/rke-clusters/node-pools/
[2]: /docs/cluster-provisioning/rke-clusters/custom-nodes/
[3]: /docs/cluster-provisioning/hosted-kubernetes-clusters/
[4]: /docs/cluster-provisioning/imported-clusters/

#### 节点池节点注意事项

使用 [其中一个节点池选项](/docs/cluster-provisioning/rke-clusters/node-pools/#node-pools) 创建的集群自动维护在初始集群配置期间设置的节点规模。这个比例决定了Rancher为集群维护的活动节点的数量。

#### 由托管的Kubernetes提供商创建的节点注意事项

在Rancher中，管理节点 [由Kubernetes提供商托管](/docs/cluster-provisioning/hosted-kubernetes-clusters/) 的选项多少有些有限。与使用Rancher UI进行编辑(如增加或减少节点数量)不同，直接编辑集群。

#### 导入节点注意事项

虽然可以使用Rancher将工作负载部署到 [导入的集群](/docs/cluster-provisioning/imported-clusters/)，但是无法管理单个集群节点。所有导入集群节点的管理都必须在Rancher之外进行。

## 隔离和驱逐节点

_隔离_ 一个节点则将其标记为不可调度。此功能对于在小型维护窗口（如重启、升级或销毁）期间在节点上执行短任务非常有用。完成后，重新打开电源，并通过取消对节点的隔离使其可以再次调度。

_驱逐_ 的过程是首先隔离节点，然后驱逐所有的pods。该功能对于执行节点维护(如内核升级或硬件维护)非常有用。它可以防止新的pods部署到节点，同时重新分配现有pods，这样用户就不会遇到服务中断。

当节点被驱逐时, pods按以下规则处理:

- 对于具有副本集的pod，该pod将被一个新的pod替换，该pod将被调度到一个新节点。另外，如果pod是服务的一部分，那么客户机将自动重定向到新的pod。

- 对于没有副本集的pod，您需要调出一个新的pod副本，并假设它不是服务的一部分，将客户端重定向到它。

您可以驱逐处于`隔离` 或者 `活动` 状态的节点。 当您驱逐一个节点时，节点被隔离，评估节点必须满足的条件来驱逐，然后(如果它满足条件)节点驱逐它的pods。

然而，您可以在初始化驱逐时覆盖驱逐条件。您还可以设置一个宽限期和超时值。

不同的Rancher版本会有不同的节点驱逐选项。

#### Rancher v2.2.x+主动的和安全的驱逐选项

两种驱逐模式：主动的和安全的。

- **主动模式**

  在这种模式下，pods不会被重新调度到一个新节点，即使它们没有控制器。Kubernetes希望您有自己的逻辑来处理删除这些pods。

  Kubernetes还希望实现能够决定如何使用emptyDir处理pods。如果pod使用emptyDir存储本地数据，您可能无法安全地删除它，因为一旦从节点删除了pod, emptyDir中的数据将被删除。选择主动模式将删除这些pods。

- **安全模式**

  如果一个节点有独立的pods或临时数据，它将被隔离，但不会被驱逐。

#### Rancher v2.2.x之前版本的主动的和安全的驱逐选项

下面的列表描述了每个驱逐选项:

- **即使存在不受ReplicationController、ReplicaSet、Job、DaemonSet或StatefulSet管理的pod **

  这些类型的pod不会被重新调度到一个新节点，因为它们没有控制器。Kubernetes希望您有自己的逻辑来处理删除这些pods。Kubernetes强迫你选择这个选项(它将会删除/驱逐这些pod)或者驱逐不会继续。

- **即使有daemonset管理的pods**

  与上面类似，如果您有任何daemonsets，则仅在选中此选项时才会进行驱逐。即使打开了这个选项，pod也不会被删除，因为它们会立即被替换。在启动时，Rancher当前在系统中有几个默认运行的守护进程集，因此该选项在默认情况下是打开的。

- **即使有使用emptyDir的pods**

  如果pod使用emptyDir存储本地数据，您可能无法安全地删除它，因为一旦从节点删除了pod, emptyDir中的数据将被删除。与第一个选项类似，Kubernetes希望实现能够决定如何处理这些pods。选择此选项将删除这些pods。

#### 宽限期

给予每个pod清理时间，这样它们就有机会优雅地离开。例如，当pods可能需要完成任何未完成的请求时，回滚事务或将状态保存到某个外部存储中。如果是负数，将使用pod中指定的默认值。

#### 超时

在放弃操作之前驱逐应该继续等待的时间。

> **Kubernetes已知问题:** 目前 [超时设置](https://github.com/kubernetes/kubernetes/pull/64378) 在驱逐节点时不强制执行。这个问题将在Kubernetes 1.12中更正。

#### 驱逐和隔离状态

如果有任何与用户输入相关的错误，节点将进入 `隔离` 状态，因为驱逐失败。您可以更正输入并尝试再次驱逐该节点，也可以通过解除该节点的隔离来中止。

如果驱逐继续没有错误，节点进入 `驱逐中` 的状态。 当节点处于这种状态时，您可以选择停止驱逐，这将停止驱逐过程并将节点的状态更改为 `已隔离`。

一旦驱逐成功完成，节点将处于`已驱逐`状态。然后可以关闭或删除节点。

> **想知道更多关于隔离和驱逐的信息吗?** 请残月 [Kubernetes文档](https://kubernetes.io/docs/tasks/administer-cluster/cluster-management/#maintenance-on-a-node).

## 编辑节点

编辑节点允许您:

- 更改它的名字
- 更改它的描述
- 添加 [标签](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
- 添加/删除 [污点](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)

## 查看节点API

选择此选项以查看节点的 [API endpoints](/docs/api/)。

## 删除节点

使用**删除** 从云提供商删除有缺陷的节点。当您删除一个有缺陷的节点时，Rancher会自动将其替换为具有相同配置的节点。

> **提示:** 如果您的集群由基础设施提供商托管，并且您希望降低集群规模，而不是删除有缺陷的节点，那么 [降低规模](#scaling-nodes) 而不是删除。

## 扩展节点

对于由基础设施提供商托管的节点，您可以使用scale控件来扩展每个节点池中的节点数量。此选项不适用于其他集群类型。

## SSH到由基础设施提供商托管的节点

对于 [由基础设施提供商托管的节点](/docs/cluster-provisioning/rke-clusters/node-pools/)，您可以选择下载它的SSH密钥，以便您可以从桌面远程连接到它。

1. 在由基础设施提供商托管的集群中，从主菜单中选择 **节点**。

1. 找到要远程进入的节点。 选择 **省略号 (...) > 下载Keys**.

   **步骤结果：** 一个包含用于SSH的文件的ZIP文件被下载。

1. 将ZIP文件解压缩到任何位置。

1. 打开终端。切换到解压缩的ZIP文件目录。

1. 输入以下命令:

   ```
   ssh -i id_rsa root@<IP_OF_HOST>
   ```

## 管理节点池

> **先决条件：** 以下选项仅适用于 [使用RKE启动](/docs/cluster-provisioning/rke-clusters/) 的集群。节点池功能不适用于导入的集群或由Kubernetes提供商托管的集群。

在 [由RKE启动](/docs/cluster-provisioning/rke-clusters/) 的集群中，你可以：

- 向集群中添加新的 [节点池](/docs/cluster-provisioning/rke-clusters/node-pools/)。添加到池中的节点是根据您使用的 [节点模版](/docs/user-settings/node-templates/) 创建的。

      	- 点击 **+** 按照屏幕上的说明创建一个新模板。

         	- 您还可以通过从 **模版** 下拉列表中选择一个来重用现有的模板。

- 通过选择不同的复选框，在节点池中重新分配Kubernetes角色。

- 增加或减少池中的节点数量 (不过，如果只是想保持节点的规模，我们建议使用集群的 [节点选项卡](/docs/k8s-in-rancher/nodes/#nodes-provisioned-by-node-pool) 选项卡)。
