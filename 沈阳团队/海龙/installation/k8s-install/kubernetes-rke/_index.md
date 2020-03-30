---
title: 2. 设置Kubernetes集群
---

本节介绍如何根据[Rancher server环境的最佳实践](/docs/overview/architecture-recommendations/#environment-for-kubernetes-installations)在您的三个节点上安装Kubernetes集群。这个集群应该专用于运行Rancher server。我们建议使用RKE在这个集群上安装Kubernetes。不应使用托管的Kubernetes提供商，例如EKS。

对于无法直接访问Internet的系统，请参阅[Air Gap：Kubernetes安装。](/docs/installation/air-gap-high-availability/)

> **单节点安装提示：**
> 在单节点Kubernetes集群中，Rancher server不具有高可用性，这对于在生产环境中运行Rancher至关重要。但是，如果要在短期内通过使用单个节点来节省资源，同时保留高可用性迁移路径，则在单节点群集上安装Rancher可能会很有用。
>
> 要设置单节点集群，在使用RKE配置集群时，只需在`cluster.yml`中配置一个节点。单个节点应该具有所有三个角色:`etcd`、`controlplane`和`worker`。这样，Rancher就可以像安装在其他集群上一样，使用Helm安装集群。

#### 创建`rancher-cluster.yml`文件

使用下面的示例，创建`rancher-cluster.yml`文件。将`nodes`列表中的IP地址替换为您创建的3个节点的IP地址或DNS名称。

如果您的节点具有公共和内部地址，建议设置`internal_address：`这样Kubernetes会将其用于集群内通信。某些服务（例如AWS EC2）需要设置`internal_address：`如果您想使用自引用安全组或防火墙。

```yaml
nodes:
  - address: 165.227.114.63
    internal_address: 172.16.22.12
    user: ubuntu
    role: [controlplane, worker, etcd]
  - address: 165.227.116.167
    internal_address: 172.16.32.37
    user: ubuntu
    role: [controlplane, worker, etcd]
  - address: 165.227.127.226
    internal_address: 172.16.42.73
    user: ubuntu
    role: [controlplane, worker, etcd]

services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 24h

## Required for external TLS termination with
## ingress-nginx v0.22+
ingress:
  provider: nginx
  options:
    use-forwarded-headers: 'true'
```

##### RKE节点选项

| 选项             | 需要 | 描述                                                                            |
| ------------------ | -------- | -------------------------------------------------------------------------------------- |
| `address`          | yes      | 公用DNS或IP地址                                                           |
| `user`             | yes      | 可以运行docker命令的用户                                                    |
| `role`             | yes      | 分配给节点的Kubernetes角色列表                                          |
| `internal_address` | no       | 内部群集流量的专用DNS或IP地址                             |
| `ssh_key_path`     | no       | 用于对节点进行身份验证的SSH私钥的路径（默认为`~/.ssh/id_rsa`） |

##### 高级配置

RKE有许多配置选项可用于自定义安装在您的特定环境。

请参阅[RKE文档]({{<baseurl>}}/rke/latest/en/config-options/)来了解RKE的选项和功能的完整列表。

要为大规模Rancher安装etcd集群，请参阅[etcd设置指南](/docs/installation/options/etcd/)。

#### 运行RKE

```
rke up --config ./rancher-cluster.yml
```

完成后，它应该以这样一行结束： `Finished building Kubernetes cluster successfully`.

#### 测试集群

RKE应该已经创建了一个文件`kube_config_rancher-cluster.yml`。该文件具有`kubectl`和`helm`的凭据。

> **注意：** 如果您使用了与`rancher-cluster.yml`不同的文件名，则kube配置文件将命名为`kube_config_<FILE_NAME>.yml`。

您可以将此文件复制到`$HOME/.kube/config`，或者如果您使用多个Kubernetes集群，请将`KUBECONFIG`环境变量设置为`kube_config_rancher-cluster.yml`的路径。

```
export KUBECONFIG=$(pwd)/kube_config_rancher-cluster.yml
```

使用`kubectl`测试您的连通性，并查看您的所有节点是否都处于`Ready`状态。

```
kubectl get nodes

NAME                          STATUS    ROLES                      AGE       VERSION
165.227.114.63                Ready     controlplane,etcd,worker   11m       v1.13.5
165.227.116.167               Ready     controlplane,etcd,worker   11m       v1.13.5
165.227.127.226               Ready     controlplane,etcd,worker   11m       v1.13.5
```

#### 检查集群pod的运行状况

检查所有必需的pod和容器是否状况良好，可以继续进行。

- Pod是`Running`或`Completed`状态。
- `READY`列显示所有正在运行的容器 (即`3/3`)，`STATUS`显示POD是`Running`。
- `STATUS` `Completed`的窗格是一次运行的作业。对于这些pod，`READY`应为`0/1`。

```
kubectl get pods --all-namespaces

NAMESPACE       NAME                                      READY     STATUS      RESTARTS   AGE
ingress-nginx   nginx-ingress-controller-tnsn4            1/1       Running     0          30s
ingress-nginx   nginx-ingress-controller-tw2ht            1/1       Running     0          30s
ingress-nginx   nginx-ingress-controller-v874b            1/1       Running     0          30s
kube-system     canal-jp4hz                               3/3       Running     0          30s
kube-system     canal-z2hg8                               3/3       Running     0          30s
kube-system     canal-z6kpw                               3/3       Running     0          30s
kube-system     kube-dns-7588d5b5f5-sf4vh                 3/3       Running     0          30s
kube-system     kube-dns-autoscaler-5db9bbb766-jz2k6      1/1       Running     0          30s
kube-system     metrics-server-97bc649d5-4rl2q            1/1       Running     0          30s
kube-system     rke-ingress-controller-deploy-job-bhzgm   0/1       Completed   0          30s
kube-system     rke-kubedns-addon-deploy-job-gl7t4        0/1       Completed   0          30s
kube-system     rke-metrics-addon-deploy-job-7ljkc        0/1       Completed   0          30s
kube-system     rke-network-plugin-deploy-job-6pbgj       0/1       Completed   0          30s
```

#### 保存文件

> **重要**
> 需要以下文件来维护，故障排除和升级群集。

将以下文件的副本保存在安全的位置：

- `rancher-cluster.yml`: RKE群集配置文件。
- `kube_config_rancher-cluster.yml`: 集群的[Kubeconfig文件]({{<baseurl>}}/rke/latest/en/kubeconfig/)，此文件包含用于访问集群的凭据。
- `rancher-cluster.rkestate`: [Kubernetes群集状态文件]({{<baseurl>}}/rke/latest/en/installation/#kubernetes-cluster-state)，此文件包含用于完全访问群集的凭据。<br/><br/>_Kubernetes集群状态文件仅在使用RKE v0.2.0或更高版本时创建。_

#### 问题或错误？

请参阅[故障排除](/docs/installation/options/troubleshooting/)页面.

#### [下一步: 安装Rancher](/docs/installation/k8s-install/helm-rancher/)
