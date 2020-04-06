---
标题: 从版本 v1.6 迁移到 v2.x
---

Rancher v2.x 经过重新整理和编写，旨在为Kubernetes和Docker提供一个完整的管理解决方案。 由于进行了较为广泛的更改，因此没有从版本v1.6直接升级到v2.x的途径，而是将v1.6服务迁移到v2.x作为Kubernetes的工作负载。 在v1.6版本中，最常用的编排是Rancher自己的引擎Cattle。 以下指南将说明和指导我们的Cattle用户如何在Kubernetes环境中运行工作负载。


### 视频

该视频演示了从Rancher v1.6到v2.x的完整迁移过程。

{{< youtube OIifcqj5Srw >}}

### 迁移计划

> **在开始之前想了解有关Kubernetes的更多信息?** 阅读我们的 [Kubernetes 介绍](/docs/v1.6-migration/kub-intro).

- [1. 开始使用](/docs/v1.6-migration/get-started)

  > **在版本v1.6时已经是Kubernetes的使用者?**
  >
  > _开始使用_ 是您为迁移到v2.x唯一需要看的部分。您可以跳过其他所有内容


- [2. 迁移您的服务](/docs/v1.6-migration/run-migration-tool/)
- [3. 暴露您的服务](/docs/v1.6-migration/expose-services/)
- [4. 配置运行状况健康检查](/docs/v1.6-migration/monitor-apps)
- [5. 调度您的服务](/docs/v1.6-migration/schedule-workloads/)
- [6. 服务发现](/docs/v1.6-migration/discover-services/)
- [7. 负载均衡](/docs/v1.6-migration/load-balancing/)

### 迁移示例文件

在整个迁移指南中，我们将引用几个要从Rancher v1.6迁移到v2.x的的示例服务。 这些服务是：


- 名为 `web` 的服务, 该服务运行 [Let's Chat](http://sdelements.github.io/lets-chat/), 一个为小团队的自托管聊天服务。
- 名为 `database` 的服务, 该服务运行 [Mongo DB](https://www.mongodb.com/), 一个开源文档数据库。
- 名为 `webLB`的服务, 该服务运行 [HAProxy](http://www.haproxy.org/), 一个Rancher v1.6中使用的开源负载均衡器。

在迁移过程中，我们将从Rancher v1.6中导出这些服务，为每个Rancher v1.6环境和堆栈导出生成一个唯一的目录，并且输出两个文件到每个堆栈的目录中:


- `docker-compose.yml`

  包含堆栈中每个服务的标准Docker指令文件。 我们将把这些文件转换为Rancher v2.x可以读取的Kubernetes清单。

- `rancher-compose.yml`

  用于Rancher特定功能的文件，例如运行健康检查和负载均衡器。 Rancher v2.x无法读取这些文件，因此不必担心它们的内容--我们正丢弃它们，然后使用v2.x UI重新创建。


#### [下一步: 开始使用](/docs/v1.6-migration/get-started)
