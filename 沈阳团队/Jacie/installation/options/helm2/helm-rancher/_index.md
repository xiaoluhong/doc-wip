---
title: '4.  安装Rancher'
---

可以使用Kubernetes的helm包管理工具来管理Rancher的安装。使用 `helm` 来可以一键安装Rancher及其依赖组件。

对于无法访问互联网的环境，请查看 [离线安装kubernetes](/docs/installation/air-gap-installation/install-rancher/)

请参阅[Helm 版本要求](/docs/installation/options/helm-version) 来选择安装Rancher的Helm版本。

> **提示:** 安装说明假定您使用的是Helm 2。安装说明将很快更新到使用Helm 3版本。 如果您想直接使用Helm 3，请参照[此说明](https://github.com/ibrokethecloud/rancher-helm3)

#### 增加helm chart仓库

使用 `helm repo add` 命令添加包含Rancher chart的Helm仓库来安装Rancher。更多信息，请查看[选择Rancher版本](/docs/installation/options/server-tags/#helm-chart-repositories)来选择最适合您的仓库。

{{< release-channel >}}

```
helm repo add rancher-<CHART_REPO> https://releases.rancher.com/server-charts/<CHART_REPO>
```

#### 选择 SSL 配置

Rancher Server默认需要SSL/TLS配置来保证访问的安全性。

以下有三种关于证书配置的建议。

> **提示:** 如果您想要将SSL/TLS访问在外部终止，请查看[使用外部TLS负载均衡器](/docs/installation/options/helm2/helm-rancher/chart-options/#external-tls-termination).

| Configuration                                                     | Chart option                     | Description                                                                                 | Requires cert-manager                 |
| ----------------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------- |
| [Rancher 自签名证书](#rancher-generated-certificates) | `ingress.tls.source=rancher`     | 使用Rancher CA签发的证书<br/>此项为 **默认选项** | [yes](#optional-install-cert-manager) |
| [Let’s Encrypt](#let-s-encrypt)                                   | `ingress.tls.source=letsEncrypt` | 使用 [Let's Encrypt](https://letsencrypt.org/)颁发的证书                        | [yes](#optional-install-cert-manager) |
| [自签名证书](#certificates-from-files)               | `ingress.tls.source=secret`      | 使用您的自签名证书（Kubernetes Secret）                             | no                                    |

#### 选项: 安装 cert-manager

**提示:** 仅由Rancher生成的CA(`ingress.tls.source=rancher`) 和Let's Encrypt颁发的证书(`ingress.tls.source=letsEncrypt`)才需要cert-manager。如果您使用自己的证书文件(option `ingress.tls.source=secret`) 或者[使用外部TLS负载均衡器](/docs/installation/options/helm2/helm-rancher/chart-options/#external-tls-termination) 可以跳过此步骤。

> **重要:**
> 由于Helm v2.12.0和cert-manager的问题，请使用Helm v2.12.1或更高版本。

> cert-manager最近的更改需要升级版本。如果您正在升级Rancher并且使用低于v0.9.1版本的cert-manager，请参考 [升级文档](/docs/installation/options/upgrading-cert-manager/).

Rancher依靠 [cert-manager](https://github.com/jetstack/cert-manager) 使用Rancher CA 生成证书或者请求Let's Encrypt签发的证书。

这些说明来自 [cert-manager 官方文档](https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html#installing-with-helm).

1. 单独安装CustomResourceDefinition资源

   ```plain
   kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.9/deploy/manifests/00-crds.yaml
   ```

1. 为cert-manager创建命名空间

   ```plain
   kubectl create namespace cert-manager
   ```

1. 标记cert-manager命名空间来禁用资源验证

   ```plain
   kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true
   ```

1. 添加 Jetstack Helm repository

   ```plain
   helm repo add jetstack https://charts.jetstack.io
   ```

1. 更新您本地的Helm chart repository缓存

   ```plain
   helm repo update
   ```

1. 使用Helm chart 安装 cert-manager 
   ```plain
   helm install \
     --name cert-manager \
     --namespace cert-manager \
     --version v0.9.1 \
     jetstack/cert-manager
   ```

安装cert-manager以后，您可以通过检查cert-manager命名空间下的pod运行状态来验证部署是否正确：

```
kubectl get pods --namespace cert-manager

NAME                                            READY   STATUS      RESTARTS   AGE
cert-manager-7cbdc48784-rpgnt                   1/1     Running     0          3m
cert-manager-webhook-5b5dd6999-kst4x            1/1     Running     0          3m
cert-manager-cainjector-3ba5cd2bcd-de332x       1/1     Running     0          3m
```

如果"webhook" pod(第二行)处于ContainerCreating状态，它可能正在等待Secret被mount到pod中。如果等待几分钟还是处于这种状态或者有其他的问题，请查看[常见问题](https://docs.cert-manager.io/en/latest/getting-started/troubleshooting.html) guide.

<br/>

##### Rancher自签名证书

> **提示:** 在执行以下操作之前，您需要安装 [cert-manager](#optional-install-cert-manager)

默认情况下Rancher生成一个私有CA并使用 `cert-manager` 来颁发证书用以访问Rancher界面。因为 `rancher` 是 `ingress.tls.source` 选项的默认值，我们在运行 `helm install` 命令的时候并没有指定 `ingress.tls.source` 选项。

- 将 `hostname` 设置为指向您负载均衡器的DNS名称

```
helm install rancher-<CHART_REPO>/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org
```

等待Rancher运行：

```
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

##### Let's Encrypt

> **提示:** 在执行以下操作之前，您需要安装[cert-manager](#optional-install-cert-manager)

该选项使用 `cert-manager` 自动请求和更新[Let's Encrypt](https://letsencrypt.org/)证书。这是一个免费的服务，它为您提供一个受信的证书，因为Let's Encrypt提供的是受信的CA。此配置使用HTTP(`HTTP-01`)验证，因此负载均衡器必须具有公共的DNS记录并可以从互联网访问到。

- 将 `hostname` 设置为公共DNS记录, 将 `ingress.tls.source` 选项设置为 `letsEncrypt`，并且设置 `letsEncrypt.email` 为可通讯的电子邮件地址，方便发送通知(例如证书到期通知)

```
helm install rancher-<CHART_REPO>/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=me@example.org
```

等待Rancher运行：

```
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

##### 自签名证书

根据您自己的证书创建Kubernetes密文以供Rancher使用。

> **提示:** 服务器证书中的 `Common Name` 或 `Subject Alternative Names` 必须与 `hostname` 选项一致, 否则ingress controller将无法正确配置。尽管技术上仅需要 `Subject Alternative Names`，拥有一个匹配的 `Common Name` 可以最大程度的提高与旧版浏览器/应用程序的兼容性。如果您想检查证书是否正确，请查看[如何在服务器证书中检查Common Name和 Subject Alternative Names？](/docs/faq/technical/#how-do-i-check-common-name-and-subject-alternative-names-in-my-server-certificate)

- 设置 `hostname` 并且将 `ingress.tls.source` 选项设置为 `secret`.
- 如果您使用的是私有CA证书，请在下面的命令中增加 `--set privateCA=true`.

```
helm install rancher-<CHART_REPO>/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=secret
```

现在已经部署完Rancher，请参见[增加TLS密文](/docs/installation/options/helm2/helm-rancher/tls-secrets/)来发布证书文件，以便Rancher与ingress controller可以使用证书。

增加完密文后，检查Rancher是否运行成功：

```
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

如果你看到了错误：`error: deployment "rancher" exceeded its progress deadline`，您可以使用下面的命令来检查deployment的状态：

```
kubectl -n cattle-system get deploy rancher
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rancher   3         3         3            3           3m
```

显示的 `DESIRED` 和 `AVAILABLE` 数量应该一致。

#### 高级配置

Rancher chart有需要配置选项可用于自定义安装Rancher来适配您的环境。以下是一些常见的高级场景。

- [HTTP Proxy](/docs/installation/options/helm2/helm-rancher/chart-options/#http-proxy)
- [私有镜像仓库](/docs/installation/options/helm2/helm-rancher/chart-options/#private-registry-and-air-gap-installs)
- [使用外部负载均衡器](/docs/installation/options/helm2/helm-rancher/chart-options/#external-tls-termination)

关于完整的Chart选项，请参见[Chart Options](/docs/installation/options/helm2/helm-rancher/chart-options/)

#### 保存配置选项

请确保已经保存了您使用 `--set` 设置的配置选项，在您使用Helm升级Rancher到新版本时需要用到相同的配置选项。

#### 总结

通过上面的操作，您应该已经完成了Rancher server的安装。在浏览器中输入您配置的域名，便可以访问Rancher的登录页面了。

还不可用？请查看[常见问题](/docs/installation/options/helm2/helm-rancher/troubleshooting/)
