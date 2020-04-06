---
标题: Kubernetes介绍
---

Rancher v2.x建立在[Kubernetes](https://kubernetes.io/docs/home/?path=users&persona=app-developer&level=foundational) 上的容器调度编排。 v2.x底层技术的这种转变与v1.6有很大的不同，后者支持了几种流行的容器编排工具。 由于Rancher现在完全基于Kubernetes，因此学习Kubernetes基础很有帮助。

下表介绍并定义了一些关键的Kubernetes概念。

| **概念**    | **定义**                                                                                                                                                                                |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 集群        | 运行Kubernetes管理的容器化应用程序的机器的集合。                                                                                                           |
| 命名空间    | 一个虚拟集群，单个物理集群可以支持多个集群。                                                                                                           |
| 节点        | 组成集群的物理机或虚拟机之一。                                                                                                                               |
| Pod         | 最小最简单的Kubernetes对象。 一个pod代表一套正在您的集群上运行的 [容器](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/#why-containers) | 
| Deployment  | 管理复制的应用程序的API对象。                                                                                                                                          |
| 工作负载    | 工作负载是为pods各项设置部署规则的对象。                                                                                                                                  |

### 迁移备忘录

由于Rancher v1.6默认为我们的的Cattle容器编排器，因此它主要使用与Cattle相关的术语。 但是，由于Rancher v2.x使用Kubernetes，因此符合Kubernetes命名标准。 对于不熟悉Kubernetes的人来说，这种转变可能会令人困惑，因此我们创建了一个表，该表将Rancher v1.6中常用的术语映射到Rancher v2.x中的等效术语。

| **Rancher v1.6** | **Rancher v2.x**                            |
| ---------------- | ------------------------------------------- |
| 容器             | Pod                                         |
| 服务             | 工作负载                                    |
| 负载均衡         | 入口                                        |
| 堆栈             | 命名空间                                     |
| 环境             | 项目 (管理)/集群 (计算)                       |
| 主机             | 节点                                         |
| 目录             | Helm                                        |
| 端口映射         | HostPort (单节点)/NodePort (多节点)           |

<br/>
有关Kubernetes概念的更多详细信息，请参见
[Kubernetes概念文档](https://kubernetes.io/docs/concepts/).

#### [下一步: 开始使用](/docs/v1.6-migration/get-started/)
