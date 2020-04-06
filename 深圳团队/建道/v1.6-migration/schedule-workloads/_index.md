---
标题: '5. 调度您的服务'
---

在v1.6中，称为_服务_的对象用于将容器调度到集群主机。 服务包括应用程序的Docker镜像以及所需状态的配置。

在Rancher v2.x中，等效对象称为_工作负载_。 Rancher v2.x保留了v1.6以来的所有调度功能，但是由于从Cattle更改为Kubernetes做默认容器编排，因此用于调度工作负载的术语和机制也发生了变化。

工作负载deployment是容器编排的更重要和更复杂的方面之一。 将pod部署到可用的共享集群资源有助于在最佳使用计算资源的情况下最大化性能。

您可以在编辑deployment时调度迁移的v1.6服务。 通过调度服务 **工作负载类型** 和 **节点调度** 部分，如下所示。

<figcaption>Editing Workloads: Workload Type and Node Scheduling Sections</figcaption>

![工作负载类型和节点调度部分](/img/rancher/migrate-schedule-workloads.png)

### 该文档

<!-- NEED DOCS ABOUT CHANGING DEPLOYMENTS TO DAEMONSETS -->

<!-- TOC -->

- [调度服务有何不同?](#whats-different-for-scheduling-services)
- [节点调度选项](#node-scheduling-options)
- [调度Pod到特定节点](#scheduling-pods-to-a-specific-node)
- [使用标签调度](#scheduling-using-labels)
- [使用资源约束调度Pod](#scheduling-pods-using-resource-constraints)
- [防止将特定服务调度到特定节点](#preventing-scheduling-specific-services-to-specific-nodes)
- [调度全局的服务](#scheduling-global-services)

<!-- /TOC -->

### 调度服务有何不同?

Rancher v2.x保留了v1.6中可用的_所有_方法来调度服务。 但是，由于默认的容器编排系统已从Cattle更改为Kubernetes，因此每个调度选项的术语和实现都已更改。

在v1.6中，您可以在将服务添加到堆栈的同时调度服务到主机。 在Rancher v2.x.中，等效的操作是为deployment调度工作负载。 下图显示了Rancher v2.x与v1.6中用于调度的UI的比较。

![节点调度: Rancher v2.x vs v1.6](/img/rancher/node-scheduling.png)

### 节点调度选项

在调度节点以托管工作负载pods时（即在Rancher v1.6中为容器调度主机），Rancher提供了多种选择。

您可以在部署工作负载时选择调度选项。 术语_工作负载_是在Rancher v1.6中将服务添加到堆栈的同义词。 您可以通过使用上下文菜单浏览到集群项目来部署工作负载 (`<CLUSTER> > <PROJECT> > Workloads`).

以下各节提供有关使用每个调度选项的信息，以及对Rancher v1.6的任何显着更改。 有关在Rancher v2.x中部署工作负载的完整说明，而不仅仅是调度选项，请参阅 [部署工作负载](/docs/k8s-in-rancher/workloads/deploy-workloads/).

| 选项                                                                                                                   | v1.6 特征 | v2.x 特征 |
| -----------------------------------------------------------------------------------------------------------------------| ------------ | ------------ |
| [调度一定数量的pods?](#schedule-a-certain-number-of-pods)                                                               | ✓            | ✓            |
| [调度pods到特定节点?](#scheduling-pods-to-a-specific-node)                                                              | ✓            | ✓            |
| [调度节点通过使用标签?](#applying-labels-to-nodes-and-pods)                                                              | ✓            | ✓            |
| [调度节点通过使用标签关联性/非关联性规则?](#label-affinity-antiaffinity)                                                  | ✓            | ✓            |
| [调度基于资源约束?](#scheduling-pods-using-resource-constraints)                                                        | ✓            | ✓            |
| [阻止将特定服务调度到特定主机?](#preventing-scheduling-specific-services-to-specific-nodes)                              | ✓            | ✓            |
| [调度全局服务?](#scheduling-global-services)                                                                            | ✓            | ✓            |

#### 调度一定数量的pods

在v1.6中，您可以控制为服务部署的容器副本数量。 您可以在v2.x中以相同的方式调度pods，但在编辑工作负载时必须手动设置比例。

![解析规模](/img/rancher/resolve-scale.png)

在迁移期间，您可以通过为下面描述的**工作负载类型**选项**可扩展 deployment**设置一个值来解析`output.txt`中的`scale`条目。

<figcaption>Scalable Deployment Option</figcaption>

![工作负载规模](/img/rancher/workload-type-option.png)

#### 调度pods到特定节点

正如您可以将容器调度到Rancher v1.6中的单个主机一样，也可以将容器调度到Rancher v2.x中的单个节点。

部署工作负载时，请使用**调度节点**部分选择要在其上运行Pod的节点。 调度下面的工作负载在特定节点上部署一个具有两个pods规模的Nginx镜像。

<!-- Question: What would be a good use case for use of a scheduling pods on the same node?-->

<figcaption>Rancher v2.x: Workload Deployment</figcaption>

![工作负载选项卡和按节点分组图标](/img/rancher/schedule-specific-node.png)

Rancher会将Pod调度到您选择的节点如果1)有可用于该节点的计算资源，并且2）您已将端口映射配置为使用HostPort选项，并且没有端口冲突。

如果使用与另一个工作负载冲突的NodePort暴露工作负载，则将成功创建deployment，但不会创建NodePort服务。 因此，工作负载不会暴露在集群外部。

创建工作负载后，您可以确认将pods调度到所选节点。 从项目视图中，单击 **资源 > 工作负载.**（在v2.3.0之前的版本中，单击 **工作负载**选项卡。）单击**节点组**图标，以按节点对工作负载进行排序。 请注意，两个Nginx Pod都调度在同一节点上。

![Pods调度到同一个节点](/img/rancher/scheduled-nodes.png)

<!--

If you export the workload's manifest for Rancher v2.x, you can see in the pod spec that the workload is scheduled to the node that you selected (`nodeName: mark-do1`).

<figcaption>Kubernetes manifest: All Pods Scheduled to Single Node</figcaption>

```YAML
...
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          name: 80tcp01
          protocol: TCP
        resources: {}
        securityContext:
          allowPrivilegeEscalation: false
          capabilities: {}
          privileged: false
          readOnlyRootFilesystem: false
          runAsNonRoot: false
        stdin: true
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        tty: true
      dnsPolicy: ClusterFirst
      nodeName: mark-do1         # Scheduled Node
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
...

```
-->

#### 使用标签调度

在Rancher v2.x中，您可以为调度到特定节点约束pods（在v1.6中称为主机）。 使用 [标签]](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/), 它们是可以附加到不同Kubernetes对象的键/值对，您可以配置工作负载，以便将标记的pod分配给特定的节点（或具有特定标签的节点自动分配给工作负载pods）。

<figcaption>Label Scheduling Options</figcaption>

| 标签对象      | Rancher v1.6 | Rancher v2.x |
| -------------| ------------ | ------------ |
| 按节点调度?   | ✓            | ✓            |
| 按Pod调度?    | ✓            | ✓            |

##### 将标签应用于节点和Pods

在基于标签调度pods之前，必须首先将标签应用于pods或节点。

> **Hooray!**
> 您在Rancher v1.6中手动应用的所有标签（但_不是_由Rancher自动创建的标签）由迁移工具CLI解析，这意味着您不必手动重新应用标签。

要将标签应用于容器，请在配置工作负载时在**标签和注释**部分中添加其内容。 完成工作负载配置后，可以通过查看调度的每个pod来查看标签。 要将标签应用于节点，请编辑您的节点并在**标签**部分中添加其内容。

##### 标签的关联性/非关联性

v1.6中一些最常用的调度功能是关联性和非关联性规则。

<figcaption><code>output.txt</code> Affinity Label</figcaption>

![关联性标签](/img/rancher/resolve-affinity.png)

- **关联性**

  共享相同标签的所有pods都调度到同一节点。 可以通过以下两种方式之一配置关联性：

  | 关联性   | 描述                                                                                                                                                                                                                                                                                                                                       |
  | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | **高** | 高关联规则意味着所选择的主机必须满足所有调度规则。 如果找不到此类主机，则工作负载将无法部署。 在Kubernetes清单中，此规则转换为`nodeAffinity`指令。<br/><br/>要使用高关联性，请使用**全部要求**部分（请参见下图）配置规则。 |
  | **低** | Rancher v1.6用户可能熟悉低关联性规则，该规则尝试按规则调度部署，但是即使任何主机都不满足该规则也可以进行deployment。<br/><br/>要使用低关联性， 使用**优先任一项**部分配置规则（请参见下图）。                                                          |

    <br/>

<figcaption>Affinity Rules: Hard and Soft</figcaption>

    ![关联性规则](/img/rancher/node-scheduling-affinity.png)

- **关联性**

  共享相同标签的任何pods都将调度到不同的节点。 换句话说，虽然关联性彼此_吸引_了一个特定的标签，而非关联性却从其自身_驱除_了一个标签，以便将pod调度到不同的节点。

  您可以使用高或低关联性来创建非关联性规则。 但是，在创建规则时，必须使用 `is not set`或`not in list` 运算符。

  对于非关联性规则，我们建议使用带有诸如`NotIn`和`DoesNotExist`之类短语的标签，因为当用户应用非关联性规则时，这些术语会更加直观。

    <figcaption>AntiAffinity Operators</figcaption>

  ![关联性](/img/rancher/node-schedule-antiaffinity.png)

有关关联性和非关联性的详细文档，请参见 [Kubernetes文档](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity).

您在UI中创建的关联性规则会更新您的工作负载，并向工作负载Kubernetes清单规范中添加pods关联性/非关联性指令。

#### 阻止将特定服务调度到特定节点

在Rancher v1.6设置中，您可以使用标签阻止将服务调度到特定节点。 在Rancher v2.x中，您可以使用本机Kubernetes调度选项来重现此行为。

在Rancher v2.x中，可以通过对节点应用_污染_来阻止将pods调度到特定节点。 除非具有特殊权限（称为_容忍_），否则不会将pods调度到受污染的节点。 容忍是一种特殊的标签，允许将pods部署到受污染的节点。 编辑工作负载时，可以使用**调度节点**部分来应用容忍。 点击**显示高级选项**。

<figcaption>Applying Tolerations</figcaption>

![容忍](/img/rancher/node-schedule-advanced-options.png)

有关更多信息，请参阅关于[污点和容忍](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/).

#### 调度全局服务

Rancher v1.6包含功能部署 [全局服务]({{< baseurl >}}/rancher/v1.6/en/cattle/scheduling/#global-service), 这些服务会将重复的容器部署到环境中的每个主机（即Rancher v2.x术语的集群中的节点）。 如果服务声明了`io.rancher.scheduler.global: 'true'`标签，则Rancher v1.6会在环境中的每个主机上调度服务容器。

<figcaption><code>output.txt</code> Global Service Label</figcaption>

![全局服务标签](/img/rancher/resolve-global.png)

在Rancher v2.x中，您可以使用 [Kubernetes守护程序](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/), 这是特定类型的工作负载 <!-- link -->). _守护程序_的功能与Rancher v1.6全局服务完全相同。 Kubernetes调度程序在集群的每个节点上部署一个pod，并在添加新节点时，只要满足工作负载的调度要求，调度程序就会在它们上面启动新pod。 此外，在v2.x中，您还可以限制将守护程序部署到具有特定标签的节点上。

要在配置工作负载时创建守护程序，请从**工作负载类型**选项中选择**在每个节点上运行一个Pod**。

<figcaption>Workload Configuration: Choose run one pod on each node to configure daemonset</figcaption>

![选择在每个节点上运行一个pod](/img/rancher/workload-type.png)

#### 使用资源约束调度Pod

在Rancher v1.6 UI中创建服务时，可以根据您选择的硬件要求将其容器调度到主机。 然后基于带宽，内存和CPU容量的容器将调度到主机。

在Rancher v2.x中，您仍然可以指定pod所需的资源。 但是，这些选项在UI中不可用。 相反，您必须编辑工作负载的清单文件以声明这些资源约束。

要声明资源限制，请编辑迁移的工作负载，然后编辑**安全 & 主机**部分。

- 要为您的pod(s)保留可用的最低硬件条件，请编辑以下部分：

  - Memory Reservation
  - CPU Reservation
  - NVIDIA GPU Reservation

- 要为您的Pod设置最大硬件条件限制，请编辑：

  - Memory Limit
  - CPU Limit

<figcaption>Scheduling: Resource Constraint Settings</figcaption>

![资源约束设置](/img/rancher/resource-constraint-settings.png)

您可以在这些规范中找到有关这些规范以及如何使用它们的更多详细信息 [Kubernetes文档](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-requests-and-limits-of-pod-and-container).

#### [下一步: 服务发现](/docs/v1.6-migration/discover-services/)
