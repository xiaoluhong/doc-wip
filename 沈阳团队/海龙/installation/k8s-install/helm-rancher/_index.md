---
title: 3. 在Kubernetes集群上安装Rancher
---

Rancher使用Kubernetes的Helm软件包管理器安装。Helm charts为Kubernetes YAML清单文档提供了模板语法。

有了Helm，我们可以创建可配置的deployments，而不只是使用静态文件。有关创建您自己的deployments目录的更多信息，请查看https://helm.sh/中的文档。

对于无法直接访问Internet的系统，请参阅[Air Gap：安装Kubernetes](/docs/installation/air-gap-installation/install-rancher/)。

选择要安装的Rancher版本，请参阅[选择Rancher版本。](/docs/installation/options/server-tags)

要选择用于安装Rancher的Helm版本，请参阅[Helm版本要求](/docs/installation/options/helm-version)。

> **注意：**安装说明假定您使用的是Helm3。有关从Helm 2的安装迁移，请参阅官方的[Helm 2到3迁移文档](https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/)。这个[章节](/docs/installation/options/helm2)提供了使用Helm 2在Kubernetes上安装Rancher的较旧安装说明，适用于无法升级到Helm 3的情况。

#### 安装Helm

Helm需要安装一个简单的CLI工具。请参阅针对特定平台的[Helm项目提供的说明](https://helm.sh/docs/intro/install/)。

#### 添加Helm Chart存储库

使用`helm repo add`命令可以添加包含要安装Rancher chart的Helm chart存储库。有关存储库选择以及最适合您的使用场景的更多信息，请参阅[选择Rancher的版本](/docs/installation/options/server-tags/#helm-chart-repositories)。

{{< release-channel >}}

```
helm repo add rancher-<CHART_REPO> https://releases.rancher.com/server-charts/<CHART_REPO>
```

#### 为Rancher创建Namespace

我们需要定义一个Namespace，在Namespace中安装由Chart创建的资源。这应该是`cattle-system`:

```
kubectl create namespace cattle-system
```

#### 选择SSL配置

Rancher Server在默认情况下是安全的，并且需要SSL/TLS配置。

证书建议使用以下三个选项。

> **注意：** 如果要从外部终止SSL/TLS，请参阅[外部负载均衡器上的TLS终止](/docs/installation/options/chart-options/#external-tls-termination)。

| 配置                  | Chart选项                     | 描述                                                                                 |需要cert-manager                 |
| ------------------------------ | -------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------- |
| Rancher生成的证书 | `ingress.tls.source=rancher`     | 使用Rancher生成的CA颁发的证书（自签名）<br/>这是**默认值** | [yes](#optional-install-cert-manager) |
| Let’s Encrypt                  | `ingress.tls.source=letsEncrypt` |使用Let's Encrypt颁发证书                                                    | [yes](#optional-install-cert-manager) |
| 证书文件        | `ingress.tls.source=secret`      | 通过创建Kubernetes Secret(s)使用您自己的证书文件                             | no                                    |

#### 可选: 安装cert-manager

Rancher依靠[cert-manager](https://github.com/jetstack/cert-manager)从Rancher自己生成的CA颁发证书或请求Let's Encrypt证书。

`cert-manager`只适用于Rancher生成的CA (`ingress.tls.source=rancher`)和Let's Encrypt颁发的证书(`ingress.tls.source=letsEncrypt`)。如果您正在使用自己的证书文件(选项`ingress.tls.source=secret`)，或者在[外部负载均衡器上使用TLS终止](/docs/installation/options/chart-options/#external-tls-termination)，则应跳过此步骤。

 accordion id="cert-manager" label="点击展开" 

> **重要：**
> 由于Helm v2.12.0和cert-manager的问题，请使用Helm v2.12.1或更高版本。

> cert-manager的最新更改需要升级。如果要升级Rancher并使用v0.11.0之前的cert-manager版本，请参阅我们的[升级文档](/docs/installation/options/upgrading-cert-manager/)。

这些说明来自[官方的cert-manager文档](https://cert-manager.io/docs/installation/kubernetes/#installing-with-helm)。

```
## 安装CustomResourceDefinition资源
kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml

> **重要：**
> 如果您正在运行Kubernetes v1.15或更低版本，则需要在上方的kubectl apply命令中添加`--validate=false`标志，否则您将在cert-manager的CustomResourceDefinition资源中收到与x-kubernetes-preserve-unknown-fields字段有关的验证错误。这是一个良性错误，是由于kubectl执行资源验证的方式造成的。

## 为cert-manager创建名称空间
kubectl create namespace cert-manager

## 添加Jetstack Helm存储库
helm repo add jetstack https://charts.jetstack.io

## 更新本地Helm chart存储库缓存
helm repo update

## 安装cert-manager Helm chart
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.12.0
```

安装cert-manager后，您可以通过检查cert-manager namespace中正在运行的Pod来验证它是否已正确部署：

```
kubectl get pods --namespace cert-manager

NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-5c6866597-zw7kh               1/1     Running   0          2m
cert-manager-cainjector-577f6d9fd7-tr77l   1/1     Running   0          2m
cert-manager-webhook-787858fcdb-nlzsq      1/1     Running   0          2m
```

 /accordion 

#### 使用Helm和您选择的证书选项安装Rancher

 tabs 
 tab "Rancher生成的证书" 

> **注意：** 在继续操作之前，您需要安装[cert-manager](#optional-install-cert-manager)。

默认情况下，Rancher生成一个CA并使用`cert-manager`颁发证书访问Rancher server。因为`rancher`是`ingress.tls.source`的默认选项，所以在运行`helm install`命令时我们没有指定`ingress.tls.source`。

- 将`hostname`设置为您指向负载均衡器的DNS名称。

```
helm install rancher rancher-<CHART_REPO>/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org
```

等待Rancher推出:

```
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

 /tab 
 tab "Let's Encrypt" 

> **注意：** 在继续操作之前，您需要安装[cert-manager](#optional-install-cert-manager)。

此选项使用`cert-manager`自动请求和续​​订[Let's Encrypt](https://letsencrypt.org/)证书。这是一项免费服务，可为您提供有效的证书，因为我们的加密是受信任的CA。此配置使用HTTP验证（`HTTP-01`），因此负载均衡器必须具有公共DNS记录并且可以从Internet访问。

- 将`hostname`设置为公共DNS记录，将`ingress.tls.source`设置为`letsEncrypt`，将`letsEncrypt.email`设置为用于与证书进行通信的电子邮件地址（例如，到期通知）

```
helm install rancher rancher-<CHART_REPO>/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=me@example.org
```

等待Rancher推出：

```
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

 /tab 
 tab "文件证书" 根据您自己的证书创建Kubernetes secrets提供Rancher使用。

> **注意：** 服务器证书中的`Common Name`或`Subject Alternative Names`必须与`hostname`选项匹配，否则ingress控制器将无法正确配置。尽管从技术上要求在`Subject Alternative Names`中输入一个条目，但具有匹配的`Common Name`可以最大程度地与旧的浏览器/应用程序兼容。如果要检查证书是否正确，请参阅[如何检查服务器证书中的Common Name和Subject Alternative Names？](/docs/faq/technical/#how-do-i-check-common-name-and-subject-alternative-names-in-my-server-certificate)

- 设置`hostname`，并将`ingress.tls.source`设置为`secret`.
- 如果使用的是Private CA签名的证书，则将`--set privateCA=true`添加到下面的命令中。

```
helm install rancher rancher-<CHART_REPO>/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=secret
```

现在已经部署了Rancher，请参阅[添加TLS Secrets](/docs/installation/options/tls-secrets/)发布证书文件，以便Rancher和ingress控制器可以使用它们。

添加secrets后，检查Rancher是否成功推出:

```
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

如果看到以下错误： `error: deployment "rancher" exceeded its progress deadline`, 您可以通过运行以下命令来检查deployment的状态：

```
kubectl -n cattle-system get deploy rancher
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rancher   3         3         3            3           3m
```

`DESIRED`和`AVAILABLE`应该显示相同的个数。
 /tab 
 /tabs 

#### 高级配置

Rancher chart有许多自定义安装选项以适应特定的环境。以下是一些常见的高级方案。

- [HTTP 代理](/docs/installation/options/chart-options/#http-proxy)
- [私有镜像仓库](/docs/installation/options/chart-options/#private-registry-and-air-gap-installs)
- [外部负载均衡器上的TLS终止](/docs/installation/options/chart-options/#external-tls-termination)

有关选项的完整列表，请参见[Chart选项](/docs/installation/options/chart-options/).

#### 保存您的选项

确保保存了使用的`--set`选项。使用Helm升级Rancher到新版本时，将需要使用相同的选项。

#### 结束

就这样，您应该具有功能正常的Rancher server。将浏览器指向您选择的主机名，您应该会看到五颜六色的登录页面。

不work?查看[故障排除](/docs/installation/options/troubleshooting/)页面。
