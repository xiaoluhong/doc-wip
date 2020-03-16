---
标题: “使用Kubectl和kubeconfig访问集群”
---

本节描述如何从Rancher UI或您的虚拟机使用kubectl操作下游Kubernetes集群。

有关如何使用kubectl的更多信息，请参阅 [Kubernetes文档：kubectl概述](https://kubernetes.io/docs/reference/kubectl/overview/).

- [在Rancher UI中使用kubectl shell访问集群](#accessing-clusters-with-kubectl-shell-in-the-rancher-ui)
- [从您的虚拟机上使用kubectl访问集群](#accessing-clusters-with-kubectl-from-your-workstation)
- [使用kubectl创建的资源的注意事项](#note-on-resources-created-using-kubectl)
- [直接使用下游集群进行身份验证](#authenticating-directly-with-a-downstream-cluster)
  - [直接连接到定义了FQDN的集群](#connecting-directly-to-clusters-with-fqdn-defined)
  - [直接连接到没有定义FQDN的集群](#connecting-directly-to-clusters-without-fqdn-defined)

#### 在Rancher UI中使用kubectl Shell访问集群

您可以通过登录Rancher并在UI中打开kubectl shell来访问和管理集群。无需进一步配置。

1. 在 **全局** 视图中，打开您希望使用kubectl访问的集群。

2. 点击 **启动kubectl**，使用打开的窗口与Kubernetes集群进行交互。

#### 从您的虚拟机上使用kubectl访问集群

本节介绍如何下载集群的kubeconfig文件，从您的虚拟机上启动kubectl，并访问下游集群。

这种访问集群的替代方法允许您使用Rancher进行身份验证，并在不使用Rancher UI的情况下管理集群。

> **先决条件:** 这些说明假定您已经创建了一个Kubernetes集群，并且kubectl已经安装在您的虚拟机上。有关安装kubectl的帮助，请参阅官方 [Kubernetes文档](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

1. 登录Rancher. 从 **全局** 视图中，打开您希望使用kubectl访问的集群。
1. 点击 **Kubeconfig 文件**.
1. 将显示的内容复制到剪贴板。
1. 将内容粘贴到本地计算机上的新文件中。 将文件移动到 `~/.kube/config`。 注意: kubectl用于kubeconfig文件存放的默认位置是 `~/.kube/config`, 但是您可以使用任何目录，并使用 `--kubeconfig` 标记来指定它，如以下命令所示:

```
kubectl --kubeconfig /custom/path/kube.config get pods
```

1. 从您的虚拟机启动kubectl。使用它与kubernetes集群进行交互。

#### 使用kubectl创建的资源注意事项

Rancher将发现和显示由 `kubectl`创建的资源。但是，这些资源可能没有关于发现的所有必要注释。如果使用Rancher UI/API对资源执行操作(例如，扩展工作负载)，这可能会由于缺少注释而触发对资源的重新创建。只有在对发现的资源执行操作时才会发生这种情况。

## 直接使用下游集群进行身份验证

本节的目的是帮助您设置另一种方法来访问 [RKE集群](/docs/cluster-provisioning/rke-clusters)。

此方法仅对启用了 [已授权的集群端点](/docs/overview/architecture/#4-authorized-cluster-endpoint) 的RKE集群可用，当Rancher创建这个RKE集群时，它会生成一个kubeconfig文件，其中包含用于访问您的集群的其他kubectl上下文。这个额外的上下文允许您使用kubectl与下游集群进行身份验证，而无需通过Rancher进行身份验证。有关授权的集群端点如何工作的详细说明，请参阅 [此页.](../ace)

我们建议，作为一种最佳实践，您应该设置此方法来访问您的RKE集群，以便万一无法连接到Rancher时，您仍然可以访问集群。

> **先决条件:** 下面的步骤假设您已经创建了一个Kubernetes集群，并按照以下步骤 [从您的虚拟机上用kubectl连接到您的集群](#accessing-clusters-with-kubectl-from-your-workstation)。

要在下载的kubeconfig文件中找到上下文的名称，请运行：

```
kubectl config get-contexts --kubeconfig /custom/path/kube.config
CURRENT   NAME                        CLUSTER                     AUTHINFO     NAMESPACE
*         my-cluster                  my-cluster                  user-46tmn
          my-cluster-controlplane-1   my-cluster-controlplane-1   user-46tmn
```

在本例中，当您将 `kubectl` 与第一个上下文 `my-cluster`一起使用时，将通过Rancher服务器对您进行身份验证。

对于第二个上下文 `my-cluster-controlplane-1`，您将使用授权的集群端点进行身份验证，直接与下游RKE集群通信。

我们建议使用具有经过授权的集群端点的负载均衡器。有关详细信息，请参阅[推荐的架构部分](/docs/overview/architecture-recommendations/#architecture-for-an-authorized-cluster-endpoint)。

现在您已经有了直接使用集群进行身份验证所需的上下文的名称，您可以在运行kubectl命令时将上下文的名称作为一个选项传递进来。这些命令将根据您的集群是否定义了FQDN而有所不同。下面几节提供了示例。

当 `kubectl` 正常工作时，它确认您可以绕过Rancher的身份验证代理访问您的集群。

如果为集群定义了FQDN，则将创建引用FQDN的单个上下文。上下文将被命名为 `<CLUSTER_NAME>-fqdn`。当您想使用 `kubectl` 来访问这个集群而不需要Rancher时，您将需要使用这个上下文。

假设kubeconfig文件位于 `~/.kube/config`:

```
kubectl --context <CLUSTER_NAME>-fqdn get nodes
```

直接引用kubeconfig文件的位置:

```
kubectl --kubeconfig /custom/path/kube.config --context <CLUSTER_NAME>-fqdn get pods
```

#### 直接连接到没有定义FQDN的集群

如果没有为集群定义FQDN，则会创建引用控制平面中每个节点的IP地址的额外上下文。每个上下文将被命名为`<CLUSTER_NAME>-<NODE_NAME>`。当您想使用`kubectl` 来访问这个集群而不需要Rancher时，您将需要使用这个上下文。

假设kubeconfig文件位于 `~/.kube/config`:

```
kubectl --context <CLUSTER_NAME>-<NODE_NAME> get nodes
```

直接引用kubeconfig文件的位置:

```
kubectl --kubeconfig /custom/path/kube.config --context <CLUSTER_NAME>-<NODE_NAME> get pods
```

