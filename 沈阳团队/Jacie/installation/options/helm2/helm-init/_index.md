---
title: '初始化Helm：安装Tiller服务'
---

Helm是Kubernetes首选的包管理工具。Helm "charts"为Kubernetes YAML文件提供了模板语法。我们可以使用Helm部署可配置的工作负载，来代替使用静态文件的方式。如果您想创建自己的私有应用商店，请参照这里[https://helm.sh/](https://helm.sh/)的说明。使用Helm前，您需要在集群安装 `tiller` 服务端组件。

对于无法访问互联网的环境，请查看[Helm - 离线安装](/docs/installation/air-gap-installation/install-rancher/#helm)获取更多安装信息。

请参阅[Helm 版本要求](/docs/installation/options/helm-version) 来选择安装Rancher的Helm版本。

> **提示:** 安装说明假定您使用的是Helm 2。安装说明将很快更新到使用Helm 3版本。 如果您想直接使用Helm 3，请参照[此说明](https://github.com/ibrokethecloud/rancher-helm3)

#### 在集群中安装Tiller

> **重要:** 由于Helm v2.12.0和cert-manager的问题，请使用Helm v2.12.1或更高版本。

Helm在您的集群中安装 `tiller` 服务来管理charts。由于RKE默认开启了RBAC，我们将使用 `kubectl` 命令创建一个 `serviceaccount` 和 `clusterrolebinding`，为 `tiller` 提供在集群中部署应用的权限。

- 在 `kube-system` 命名空间中创建 `ServiceAccount`。
- 为 `tiller` 创建 `ClusterRoleBinding` 获取访问集群的权限。
- 最后使用 `helm` 来安装 `tiller` 服务。

```plain
kubectl -n kube-system create serviceaccount tiller

kubectl create clusterrolebinding tiller \
  --clusterrole=cluster-admin \
  --serviceaccount=kube-system:tiller

helm init --service-account tiller

## 对于中国的用户: 您需要指定特定的tiller-image来初始化tiller。
## tiller images 的tags可以从这里获取: https://dev.aliyun.com/detail.html?spm=5176.1972343.2.18.ErFNgC&repoId=62085.
## 初始化tiller时，需要通过 --tiller-image 指定tiller image

helm init --service-account tiller \
--tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:<tag>
```

> **提示:** 以上`tiller`安装后，对集群有完全访问权限，如果该集群是Rancher服务专用集群，则没有问题。请查看[helm docs](https://docs.helm.sh/using_helm/#role-based-access-control) 按照您的安全需求，限制 `tiller` 访问权限。

#### 测试Tiller

在您的集群上运行以下命令来验证 `tiller` 是否安装成功：

```
kubectl -n kube-system  rollout status deploy/tiller-deploy
Waiting for deployment "tiller-deploy" rollout to finish: 0 of 1 updated replicas are available...
deployment "tiller-deploy" successfully rolled out
```

并且运行以下命令验证Helm是否可以与 `tiller` 服务通信：

```
helm version
Client: &version.Version{SemVer:"v2.12.1", GitCommit:"02a47c7249b1fc6d8fd3b94e6b4babf9d818144e", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.12.1", GitCommit:"02a47c7249b1fc6d8fd3b94e6b4babf9d818144e", GitTreeState:"clean"}
```

#### 问题或错误？

请查看[常见问题](/docs/installation/options/helm2/helm-init/troubleshooting/)

#### [下一章: Rancher 安装](/docs/installation/options/helm2/helm-rancher/)
