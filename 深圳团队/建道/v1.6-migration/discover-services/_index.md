---
标题: '6. 服务发现'
---

服务发现是任何基于容器环境的核心功能之一。 一旦打包并启动应用程序后，下一步就是使你的环境或外部环境中的其他容器可以发现它。本文档将描述如何使用Rancher v2.x提供的服务发现支持，让你可以用名称找到它们。

本文档还将向你展示当迁移到Rancher v2.x时如何链接工作负载和服务。 使用迁移工具CLI从v1.6解析服务时，它将为每个服务输出两个文件：一个部署清单和一个服务清单。 你必须将这两个文件链接在一起，这样部署才能在v2.x中正常运行。


<figcaption>Resolve the <code>output.txt</code> Link Directive</figcaption>

![解析链接指令](/img/rancher/resolve-links.png)

### 在该文档

<!-- TOC -->

- [服务发现：Rancher v1.6 对比 v2.x](#service-discovery-rancher-v1-6-vs-v2-x)
- [命名空间内和跨命名空间的服务发现](#service-discovery-within-and-across-namespaces)
- [容器发现](#container-discovery)
- [服务别名创建](#service-name-alias-creation)

<!-- /TOC -->

### 服务发现: Rancher v1.6 vs. v2.x

对于Rancher v2.x，我们已将v1.6中使用的Rancher DNS微服务替换为本地 [Kubernetes DNS 支持](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), 它为Kubernetes工作负载和pods提供了等效的服务发现。 前Cattle用户可以在v2.x中复制Rancher v1.6中的所有服务发现功能。 并且不会有功能损失。

Kubernetes在集群中调度DNS容器和服务，这类似于 [Rancher v1.6 DNS 微服务]({{< baseurl >}}/rancher/v1.6/en/cattle/internal-dns-service/#internal-dns-service-in-cattle-environments). Kubernetes将其kubelet配置为将所有DNS查找路由到skyDNS服务，这是默认Kube-DNS实现的一种形式。

下表显示了两个Rancher版本中可用的每个服务发现功能。

| 服务发现功能                                       | Rancher v1.6 | Rancher v2.x | 描述                                                                                                      |
| --------------------------------------------------------------- | ------------ | ------------ | ---------------------------------------------------------------------------------------------------------------- |
| [堆栈内部和堆栈之间的服务发现][1] (i.e., clusters) | ✓            | ✓            | 堆栈内的所有服务均可通过以下方式 `<service_name>` 和 `<service_name>.<stack_name>` 跨栈。 |
| [容器发现][2]                                        | ✓            | ✓            | 所有容器都可以通过其名称全局可见。                                                            |
| [服务别名创建][3]                                | ✓            | ✓            | 向服务添加别名，并使用别名链接到其他服务。                                    |
| [发现外部服务][4]                             | ✓            | ✓            | 指向在Rancher外部部署的服务并使用外部IP或域名。                      |

[1]: #堆栈内部和堆栈之间的服务发现
[2]: #容器发现
[3]: #服务别名创建
[4]: #发现外部服务

<br/>

#### 命名空间内和跨命名空间的服务发现

在v2.x中创建_新_工作负载时 (没有迁移，更多内容 [下面](#linking-migrated-workloads-and-services)), Rancher会自动创建一个具有相同名称的服务，然后将服务和工作负载链接在一起。 如果未显式公开端口，则使用默认端口`42`。 这种做法使工作负载可以通过名称在命名空间内和命名空间之间被发现。


#### 容器发现

在Kubernetes集群中运行的各个pod也将获得已分配的用点表示法的DNS记录: `<POD_IP_ADDRESS>.<NAMESPACE_NAME>.pod.cluster.local`. 例如, 一个 IP 是 `10.42.2.7`的 pod 在 DNS 名称为 `cluster.local` 的命名空间 `default` 中将会有一个条目 `10-42-2-7.default.pod.cluster.local`.

如果在pods规范中进行了设置，也可以使用hostname和subdomain字段解析pods。 有关此解析的详细信息，请参见 [Kubernetes 文档](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/).

#### 链接迁移的工作负载和服务

当将v1.6服务迁移到v2.x时，Rancher不会自动为每个迁移的deployment创建Kubernetes服务记录。相反，你必须使用下面列出的一些方法将deployment和service手动链接在一起。

在下图中,  `web-deployment.yml` 和 `web-service.yml` 文件 [解析后创建](/docs/v1.6-migration/run-migration-tool/#migration-example-file-output) 我们的[迁移示例服务](/docs/v1.6-migration/#migration-example-files) 链接在一起。

<figcaption>Linked Workload and Kubernetes Service</figcaption>

![链接的工作负载和Kubernetes服务](/img/rancher/linked-service-workload.png)

#### 服务别名创建

正如你可以为Rancher v1.6服务创建别名一样，你也可以为Rancher v2.x工作负载执行相同的操作。 同样，你也可以使用主机名或IP地址创建指向外部运行的服务的DNS记录。 这些DNS记录是Kubernetes服务对象。

使用v2.x UI，用上下文菜单导航至 `Project` 视图. 然后单击 **资源 > 工作负载 > 服务发现.** (在v2.3.0之前的版本中, 单击 **工作负载 > 服务发现** 选项卡.) 为工作负载创建的所有现有DNS记录均列在每个命名空间下。

单击 **添加记录** 以创建新的DNS记录。支持链接到外部服务或为另一个工作负载，DNS记录或Pod组创建别名。

<figcaption>Add Service Discovery Record</figcaption>
![添加服务发现记录](/img/rancher/add-record.png)

下表指出了哪些别名选项由Kubernetes本地实现，哪些选项由Rancher利用Kubernetes实现。

| 选项                                         | Kubernetes实现?         | Rancher实现? |
| ---------------------------------------------| ----------------------- | -------------------- |
| 指向外部主机名                                | ✓                       |                      |
| 指向与选择器匹配的一组pod                      | ✓                       |                      |
| 指向外部IP地址                                |                         | ✓                    |
| 指向另一个工作负载                            |                         | ✓                    |
| 为另一个DNS记录创建别名                       |                         | ✓                    |

#### [下一步: 负载均衡](/docs/v1.6-migration/load-balancing/)
