---
title: 升级Cert-Manager
---

Rancher使用cert-manager为Rancher的HA部署自动生成和更新TLS证书。截至2019秋季，cert-manager将发生三个重要的变化，如果您有一个Rancher的HA部署，您需要采取以下措施：

1. [从2019年11月1日开始，Let's Encrypt将阻止版本低于0.8.0的cert-manager实例。](https://community.letsencrypt.org/t/blocking-old-cert-manager-versions/98753)
1. [Cert-manager正在弃用并替换certificate.spec.acme.solvers字段。](https://cert-manager.io/docs/installation/upgrading/upgrading-0.7-0.8/)此更改没有确切的截止日期。
1. [Cert-manager正在弃用`v1alpha1`API并替换它的API组。](https://cert-manager.io/docs/installation/upgrading/upgrading-0.10-0.11/)

为了解决这些变化，本指南将做两件事：

1. 记录升级cert-manager的过程
1. 解释cert-manager API的更改，并链接到cert-manager的官方文档以迁移数据

> **重要提示：**
> 如果您当前正在运行版本低于v0.11的cert-manger，并且想要将Rancher和cert-manager都升级到新版本，则需要重新安装它们：

> 1. 对运行Rancher服务器的Kubernetes集群进行一次性快照
> 2. 卸载Rancher，cert-manager和cert-manager的CustomResourceDefinition
> 3. 安装更新版本的Rancher和cert-manager

> 原因是当Helm升级Rancher时，如果运行的Rancher应用程序与用于安装它的chart模板不匹配，它将拒绝升级并显示错误消息。因为cert-manager更改了它的API组，并且我们不能修改Rancher的已发布的chart，所以cert-manager的API版本始终不匹配，因此升级将被拒绝。

> 要使用Helm重新安装Rancher，请在升级Rancher部分下选中[选项B: 重新安装Rancher Chart](/docs/upgrades/upgrades/ha/#c-upgrade-rancher)。

### 仅升级Cert-Manager

> **注意：**
> 如果您没有升级Rancher的计划，这些说明是适用的。

这些说明中使用的命名空间取决于当前安装了cert-manager的命名空间。如果它在kube-system中，请在以下说明中使用。您可以通过运行`kubectl get pods --all-namespaces`来验证，并检查cert-manager-\* pods列在哪个名称空间中。请勿更改正在运行cert-manager的名称空间，否则可能会导致问题。

> 这些说明已针对Helm 3进行了更新。如果您仍在使用Helm 2，请参阅[以下说明](/docs/installation/options/upgrading-cert-manager/helm-2-instructions)。

要升级cert-manager，请遵循以下说明：

 accordion id="normal" label="通过Internet访问升级cert-manager"

1. [备份现有资源](https://cert-manager.io/docs/tutorials/backup/)

   ```plain
   kubectl get -o yaml --all-namespaces \
   issuer,clusterissuer,certificates,certificaterequests > cert-manager-backup.yaml
   ```

   > **重要提示：**
   > 如果要从0.11.0之前的版本升级，请将所有备份资源上的apiVersion从`certmanager.k8s.io/v1alpha1`更新为`cert-manager.io/v1alpha2`。如果在其他资源上使用cert-manager注释，则需要对其进行更新以反映新的API组。有关详细信息, 请参阅有关文档[附加注释更改](https://cert-manager.io/docs/installation/upgrading/upgrading-0.10-0.11/#additional-annotation-changes)。

1. [卸载现有部署](https://cert-manager.io/docs/installation/uninstall/kubernetes/#uninstalling-with-helm)

   ```plain
   helm delete --purge cert-manager
   ```

   使用指向安装的版本vX.Y的链接删除CustomResourceDefinition

   ```plain
   kubectl delete -f https://raw.githubusercontent.com/jetstack/cert-manager/release-X.Y/deploy/manifests/00-crds.yaml
   ```

1. 单独安装CustomResourceDefinition资源

   ```plain
   kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml
   ```

   > **注意：**
   > 如果您正在运行Kubernetes v1.15或更低版本，则需要将`--validate=false`标志添加到上面的`kubectl apply`命令中。否则，您将收到一个与cert-manager的CustomResourceDefinition资源中的`x-kubernetes-preserve-unknown-fields`字段相关的验证错误。这是一个良性错误，是由于kubectl执行资源验证的方式造成的。

1. 根据需要为cert-manager创建命名空间

   ```plain
   kubectl create namespace cert-manager
   ```

1. 添加Jetstack Helm仓库

   ```plain
   helm repo add jetstack https://charts.jetstack.io
   ```

1. 更新本地Helm chart仓库缓存

   ```plain
   helm repo update
   ```

1. 安装新版本cert-manager

   ```plain
   helm install \
     cert-manager jetstack/cert-manager \
     --namespace cert-manager \
     --version v0.12.0
   ```

1. [恢复备份资源](https://cert-manager.io/docs/tutorials/backup/#restoring-resources)

   ```plain
   kubectl apply -f cert-manager-backup.yaml
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
   helm fetch jetstack/cert-manager --version v0.12.0
   ```

1. 使用您要用于安装chart的选项来渲染cert-manager模板。记得要为您从私有注册表中拉取的镜像设置`image.repository`选项。 这将创建一个带有Kubernetes manifest文件的`cert-manager`目录。

   Helm 3命令如下：

   ```plain
   helm template cert-manager ./cert-manager-v0.12.0.tgz --output-dir . \
   --namespace cert-manager \
   --set image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-controller
   --set webhook.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-webhook
   --set cainjector.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-cainjector
   ```

   Helm 2命令如下：

   ```plain
   helm template ./cert-manager-v0.12.0.tgz --output-dir . \
   --name cert-manager --namespace cert-manager \
   --set image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-controller
   --set webhook.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-webhook
   --set cainjector.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-cainjector
   ```

1. 下载cert-manager所需的CRD文件（旧的和新的）

   ```plain
   curl -L -o cert-manager/cert-manager-crd.yaml https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml
   curl -L -o cert-manager/cert-manager-crd-old.yaml https://raw.githubusercontent.com/jetstack/cert-manager/release-X.Y/deploy/manifests/00-crds.yaml
   ```

#### 安装cert-manager

1. 备份现有资源

   ```plain
   kubectl get -o yaml --all-namespaces \
   issuer,clusterissuer,certificates,certificaterequests > cert-manager-backup.yaml
   ```

   > **重要提示：**
   > 如果要从0.11.0之前的版本升级，请将所有备份资源上的apiVersion从`certmanager.k8s.io/v1alpha1`更新为`cert-manager.io/v1alpha2`。如果在其他资源上使用cert-manager注释，则需要对其进行更新以反映新的API组。有关详细信息, 请参阅有关文档[附加注释更改](https://cert-manager.io/docs/installation/upgrading/upgrading-0.10-0.11/#additional-annotation-changes)。

1. 删除现有的cert-manager安装包

   ```plain
   kubectl -n cert-manager \
   delete deployment,sa,clusterrole,clusterrolebinding \
   -l 'app=cert-manager' -l 'chart=cert-manager-v0.5.2'
   ```

   使用指向安装的版本vX.Y的链接删除CustomResourceDefinition

   ```plain
   kubectl delete -f cert-manager/cert-manager-crd-old.yaml
   ```

1. 单独安装CustomResourceDefinition资源

   ```plain
   kubectl apply -f cert-manager/cert-manager-crd.yaml
   ```

   > **注意：**
   > 如果您正在运行Kubernetes v1.15或更低版本，则需要将 `--validate=false` 标志添加到上面的 `kubectl apply` 命令中。否则，您将收到一个与cert-manager的CustomResourceDefinition资源中的 `x-kubernetes-preserve-unknown-fields` 字段相关的验证错误。这是一个良性错误，是由于kubectl执行资源验证的方式造成的。

1. 为cert-manager创建命名空间

   ```plain
   kubectl create namespace cert-manager
   ```

1. 安装cert-manager

   ```plain
   kubectl -n cert-manager apply -R -f ./cert-manager
   ```

1. [恢复备份资源](https://cert-manager.io/docs/tutorials/backup/#restoring-resources)

   ```plain
   kubectl apply -f cert-manager-backup.yaml
   ```

 /accordion 

安装了cert-manager之后，可以通过检查kube-system命名空间中运行的Pod来验证它是否已正确部署：

```
kubectl get pods --namespace cert-manager

NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-5c6866597-zw7kh               1/1     Running   0          2m
cert-manager-cainjector-577f6d9fd7-tr77l   1/1     Running   0          2m
cert-manager-webhook-787858fcdb-nlzsq      1/1     Running   0          2m
```

### Cert-Manager API更改和数据迁移

Cert-manager已经弃用 `certificate.spec.acme.solvers` 字段，并将在即将发布的版本中完全放弃对该字段的支持。

根据cert-manager文档，在v0.8中引入了配置ACME证书资源的新格式。具体来说，移动了challenge solver配置字段。从v0.9开始支持旧格式和新格式，但是在即将发布的cert-manager中将不再支持旧格式。cert-manager文档强烈建议在升级之后将ACME颁发者和证书资源更新为新格式。

有关更改和迁移说明的详细信息，请参见[cert-manager v0.7至v0.8升级说明](https://cert-manager.io/docs/installation/upgrading/upgrading-0.7-0.8/)。

v0.11版本标志着删除了先前版本的cert-manager中使用的v1alpha1 API，并且我们的API组已更改为cert-manager.io而不是certmanager.k8s.io。

我们还删除了对v0.8版本中不支持的旧配置格式的支持，这意味着在升级到v0.11之前，您必须转换到为ACME发行者使用新的solvers样式配置格式。有关更多信息，请参见[升级到v0.8指南](https://cert-manager.io/docs/installation/upgrading/upgrading-0.7-0.8/)。

有关更改和迁移说明的详细信息，请参见[cert-manager v0.10至v0.11升级说明](https://cert-manager.io/docs/installation/upgrading/upgrading-0.10-0.11/)。

有关更多信息，请参见[cert-manager升级信息](https://cert-manager.io/docs/installation/upgrading/)。
