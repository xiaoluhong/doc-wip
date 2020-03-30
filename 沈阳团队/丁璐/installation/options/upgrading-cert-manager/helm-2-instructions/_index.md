---
title: 使用Helm 2升级Cert-Manager
---

Rancher使用cert-manager为Rancher的HA部署自动生成和更新TLS证书。截至2019秋季，cert-manager将发生三个重要的变化，如果你有一个Rancher的HA部署，您需要采取以下措施：

1. [从2019年11月1日开始，Let's Encrypt将阻止版本低于0.8.0的cert-manager实例。](https://community.letsencrypt.org/t/blocking-old-cert-manager-versions/98753)
1. [Cert-manager正在弃用并替换certificate.spec.acme.solvers字段。](https://docs.cert-manager.io/en/latest/tasks/upgrading/upgrading-0.7-0.8.html#upgrading-from-v0-7-to-v0-8)此更改没有确切的截止日期。
1. [Cert-manager正在弃用`v1alpha1`API并替换它的API组。](https://cert-manager.io/docs/installation/upgrading/upgrading-0.10-0.11/)

为了解决这些变化，本指南将做两件事：

1. 记录升级cert-manager的过程
1. 解释cert-manager API的更改，并链接到cert-manager的官方文档以迁移数据

> **重要提示：**
> 如果您当前正在运行版本低于v0.11的cert-manger，并且想要将Rancher和cert-manager都升级到新版本，则需要重新安装它们：

> 1. 对运行Rancher服务器的Kubernetes集群进行一次性快照
> 2. 卸载Rancher，cert-manager和cert-manager的CustomResourceDefinition
> 3. 安装更新版本的Rancher和cert-manager

> 原因是当Helm升级Rancher时，如果运行的Rancher应用程序与用于安装它的chart模板不匹配，它将拒绝升级并显示错误消息。原因是cert-manager更改了它的API组，并且我们不能修改Rancher的已发布的chart，所以cert-manager的API版本始终不匹配，因此升级将被拒绝。

> 要使用Helm重新安装Rancher，请在升级Rancher部分下选中[选项B: 重新安装Rancher Chart](/docs/upgrades/upgrades/ha/#c-upgrade-rancher)。

### 仅升级Cert-Manager

> **注意：**
> 如果你没有升级Rancher的计划，这些说明是适用的。

这些说明中使用的命名空间取决于当前安装了cert-manager的命名空间。如果它在kube-system中，请在以下说明中使用。您可以通过运行`kubectl get pods --all-namespaces`来验证，并检查cert-manager-\* pods列在哪个名称空间中。请勿更改正在运行cert-manager的名称空间，否则可能会导致问题。

要升级cert-manager，请遵循以下说明：

 accordion id="normal" label="通过Internet访问升级cert-manager"

1.  备份现有资源

    ```plain
    kubectl get -o yaml --all-namespaces issuer,clusterissuer,certificates > cert-manager-backup.yaml
    ```

1.  删除现有部署

    ```plain
    helm delete --purge cert-manager
    ```

1.  单独安装CustomResourceDefinition资源

    ```plain
    kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.9/deploy/manifests/00-crds.yaml
    ```

1.  标记kube-system命名空间以禁用资源验证

    ```plain
    kubectl label namespace kube-system certmanager.k8s.io/disable-validation=true
    ```

1.  添加Jetstack Helm仓库

    ```plain
    helm repo add jetstack https://charts.jetstack.io
    ```

1.  更新本地Helm chart仓库缓存

    ```plain
    helm repo update
    ```

1.  安装新版本cert-manager

        ```plain
        helm install --version 0.9.1 --name cert-manager --namespace kube-system jetstack/cert-manager
        ```

     /accordion 

 accordion id="airgap" label="在airgap环境升级cert-manager"

#### 先决条件

在执行升级之前，您必须通过将必要的容器镜像添加到私有注册表中并下载或渲染所需的Kubernetes manifest文件来准备airgap环境。

1. 按照指南[准备私有注册表](/docs/installation/air-gap-installation/prepare-private-reg/)准备升级所需的镜像。

1. 从连接到Internet的系统中，将cert-manager存储库添加到Helm

   ```plain
   helm repo add jetstack https://charts.jetstack.io
   helm repo update
   ```

1. 从[Helm chart仓库](https://hub.helm.sh/charts/jetstack/cert-manager)中获取最新的cert-manager chart

   ```plain
   helm fetch jetstack/cert-manager --version v0.9.1
   ```

1. 使用您要用于安装chart的选项来渲染cert-manager模板。记得要为您从私有注册表中拉取的镜像设置`image.repository`选项。 这将创建一个带有Kubernetes manifest文件的`cert-manager`目录。

   ```plain
   helm template ./cert-manager-v0.9.1.tgz --output-dir . \
   --name cert-manager --namespace kube-system \
   --set image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-controller
   --set webhook.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-webhook
   --set cainjector.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-cainjector
   ```

1. 下载cert-manager所需的CRD文件

   ```plain
   curl -L -o cert-manager/cert-manager-crd.yaml https://raw.githubusercontent.com/jetstack/cert-manager/release-0.9/deploy/manifests/00-crds.yaml
   ```

#### 安装cert-manager

1.  备份现有资源

    ```plain
    kubectl get -o yaml --all-namespaces issuer,clusterissuer,certificates > cert-manager-backup.yaml
    ```

1.  删除现有的cert-manager安装包

    ```plain
    kubectl -n kube-system delete deployment,sa,clusterrole,clusterrolebinding -l 'app=cert-manager' -l 'chart=cert-manager-v0.5.2'
    ```

1.  单独安装CustomResourceDefinition资源

    ```plain
    kubectl apply -f cert-manager/cert-manager-crd.yaml
    ```

1.  标记kube-system命名空间以禁用资源验证

    ```plain
    kubectl label namespace kube-system certmanager.k8s.io/disable-validation=true
    ```

1.  安装cert-manager

        ```plain
        kubectl -n kube-system apply -R -f ./cert-manager
        ```

     /accordion 

安装了cert-manager之后，可以通过检查kube-system命名空间中运行的Pod来验证它是否已正确部署：

```
kubectl get pods --namespace kube-system

NAME                                            READY   STATUS      RESTARTS   AGE
cert-manager-7cbdc48784-rpgnt                   1/1     Running     0          3m
cert-manager-webhook-5b5dd6999-kst4x            1/1     Running     0          3m
cert-manager-cainjector-3ba5cd2bcd-de332x       1/1     Running     0          3m
```

如果"webhook" pod(第二行)处于ContainerCreating状态，则它可能仍在等待Secret被安装到pod中。等几分钟，以免发生这种情况，但是如果遇到问题，请查看cert-manager[故障排除](https://docs.cert-manager.io/en/latest/getting-started/troubleshooting.html)指南。

> **注意：** 上面的说明要求您将disable-validation标签添加到kube-system命名空间中。这里有一些额外的资源可以解释为什么这是必要的：
>
> - [关于disable-validation标签的信息](https://docs.cert-manager.io/en/latest/tasks/upgrading/upgrading-0.4-0.5.html?highlight=certmanager.k8s.io%2Fdisable-validation#disabling-resource-validation-on-the-cert-manager-namespace)
> - [关于证书的webhook验证的信息](https://docs.cert-manager.io/en/latest/getting-started/webhook.html)

### Cert-Manager API更改和数据迁移

Cert-manager已经弃用`certificate.spec.acme.solvers`字段，并将在即将发布的版本中完全放弃对该字段的支持。

根据cert-manager文档，在v0.8中引入了配置ACME证书资源的新格式。具体来说，移动了challenge solver配置字段。从v0.9开始支持旧格式和新格式，但是在即将发布的cert-manager中将不再支持旧格式。cert-manager文档强烈建议在升级之后将ACME颁发者和证书资源更新为新格式。

有关更改和迁移说明的详细信息，请参见[cert-manager v0.7至v0.8升级说明](https://cert-manager.io/docs/installation/upgrading/upgrading-0.7-0.8/)。

v0.11版本标志着删除了先前版本的cert-manager中使用的v1alpha1 API，并且我们的API组已更改为`cert-manager.io`而不是`certmanager.k8s.io`。

我们还删除了对v0.8版本中不支持的旧配置格式的支持，这意味着在升级到v0.11之前，您必须转换到为ACME发行者使用新的solvers样式配置格式。有关更多信息，请参见[升级到v0.8指南](https://cert-manager.io/docs/installation/upgrading/upgrading-0.7-0.8/).

有关更改和迁移说明的详细信息，请参见[cert-manager v0.10至v0.11升级说明](https://cert-manager.io/docs/installation/upgrading/upgrading-0.10-0.11/)。

有关从所有其他版本的cert-manager升级的信息，请参阅[官方文档](https://cert-manager.io/docs/installation/upgrading/)。
