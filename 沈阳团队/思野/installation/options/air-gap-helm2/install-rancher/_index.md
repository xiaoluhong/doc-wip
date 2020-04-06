---
title: 4. 安装 Rancher
---

本节介绍如何为离线安装Rancher准备您的节点。Rancher服务器可能会离线安装在防火墙或代理之后的封闭环境。这里有两个选项卡，用于高可用性（推荐）或Docker安装。

 tabs tab "Kubernetes Install 安装 (推荐)"

Rancher 建议在 Kubernetes 集群上安装 Rancher server。 Rancher server运行在由三个节点组成的高可用性的 Kubernetes 集群上.持久层(etcd)也安装在这三个节点上，以保证在其中一个节点出现错误时提供重复数据删除和数据备份。

本节描述安装 Rancher 分为五个部分：

- [A. 添加 Helm Chart 仓库](#a-add-the-helm-chart-repository)
- [B. 选择 SSL 配置](#b-choose-your-ssl-configuration)
- [C. 配置 Rancher Helm template](#c-render-the-rancher-helm-template)  
- [D. 安装 Rancher](#d-install-rancher)
- [E. Rancher versions v2.3.0 之前的版本, 配置 System Charts](#e-for-rancher-versions-prior-to-v2-3-0-configure-system-charts)

#### A. 添加Helm Chart仓库

从一个可以访问互联网的系统中， 下载最新的 Helm chart 并将其复制到一个可以访问Rancher server集群的系统中。

1. 如果还没有初始化helm,可以在有网络访问的工作站上初始化`Helm`。注意: 请参考[Helm version版本要求](/docs/installation/options/helm-version) 选择 Helm 版本来安装 Rancher。

   ```plain
   helm init -c
   ```

2. 使用`helm repo add`命令添加 Helm chart 存储库，其中包含要安装的 Rancher chart。 有关存储库选择以及哪个最适合您的用例的详细信息，请参阅[选择 Rancher 的版本](/docs/installation/options/server-tags/#helm-chart-repositories)。
   {{< release-channel >}}

   ```
   helm repo add rancher-<CHART_REPO> https://releases.rancher.com/server-charts/<CHART_REPO>
   ```

3. 获取最新的 Rancher chart 这将下载 Rancher chart 并将其以 `.tgz` 文件形式保存在当前工作目录。

```plain
helm fetch rancher-<CHART_REPO>/rancher
```

> 需要其他选项吗? 需要帮助排除故障? 请参阅 [Kubernetes Install: Advanced Options](/docs/installation/k8s-install/helm-rancher/#advanced-configurations)。

#### B. 选择 SSL 配置

Rancher server 默认设计为安全的，需要使用 SSL / TLS 配置。

当Rancher安装在离线环境下的 Kubernetes 集群时， 有两个推荐的证书源选项。

> **注意:** 如果您想在外部终结 TLS/SSL 会话，请参阅[TLS termination on an External Load Balancer](/docs/installation/options/chart-options/#external-tls-termination)。

| Configuration                              | Chart option                 | Description                                                                                                                                                   | Requires cert-manager |
| ------------------------------------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| Rancher Generated Self-Signed Certificates | `ingress.tls.source=rancher` | Use certificates issued by Rancher's generated CA (self signed)<br /> This is the **default** and does not need to be added when rendering the Helm template. | yes                   |
| Certificates from Files                    | `ingress.tls.source=secret`  | Use your own certificate files by creating Kubernetes Secret(s). <br /> This option must be passed when rendering the Rancher Helm template.                  | no                    |

#### C. 配置 Rancher Helm template

设置 Rancher Helm Template 时有几个选项是专门为离线部署设计的。

| Chart Option            | Chart Value                      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ----------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `certmanager.version`   | {`<version>`}                    | Configure proper Rancher TLS issuer depending of running cert-manager version.                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `systemDefaultRegistry` | `<REGISTRY.YOURDOMAIN.COM:PORT>` | Configure Rancher server to always pull from your private registry when provisioning clusters.                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `useBundledSystemChart` | `true`                           | Configure Rancher server to use the packaged copy of Helm system charts. The [system charts](https://github.com/rancher/system-charts) repository contains all the catalog items required for features such as monitoring, logging, alerting and global DNS. These [Helm charts](https://github.com/rancher/system-charts) are located in GitHub, but since you are in an air gapped environment, using the charts that are bundled within Rancher is much easier than setting up a Git mirror. _Available as of v2.3.0_ |

根据您在 [B. 中所做的选择](#b-choose-your-ssl-configuration) ，选择您的 SSL 配置，完成以下步骤之一。

 accordion id="self-signed" label="Option A-Default Self-Signed Certificate" 

默认情况下，Rancher 生成一个 CA 并使用 cert-manager 颁发访问 Rancher server接口的证书。

> **注意:**
> 最近证书管理器的更改需要升级。 如果您正在升级 Rancher 并且使用的 cert-manager 版本早于 v0.11.0，请参阅[我们的升级 cert-manager 文档](/docs/installation/options/upgrading-cert-manager/)。

1. 从一个可以访问外网的系统中, 将cert-manager 添加到 helm。

   ```plain
   helm repo add jetstack https://charts.jetstack.io
   helm repo update
   ```

1. 从[Helm chart repository](https://hub.helm.sh/charts/jetstack/cert-manager)拉取最新的cert-manager chart。

   ```plain
   helm fetch jetstack/cert-manager --version v0.12.0
   ```

1. 使用您用来配置cert manager template的选项来安装这个chart。记住要设置 `image.repository` 选项. 以便从您的私有仓库中提取 image。这将创建一个 cert-manager 带有 Kubernetes manifest 文件的目录。

   ```plain
   helm template ./cert-manager-v0.12.0.tgz --output-dir . \
       --name cert-manager --namespace cert-manager \
       --set image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-controller
       --set webhook.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-webhook
       --set cainjector.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-cainjector
   ```

1. 下载 cert-manager 所需的 CRD 文件
   ```plain
   curl -L -o cert-manager/cert-manager-crd.yaml https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml
   ```
1. 渲染 Rancher 模板，声明您选择的选项。 使用下面的参考表替换每个占位符。 为了配置任何由 Rancher 启动的 Kubernetes 集群或 Rancher 工具，需要将 Rancher 配置使用专用注册表。


    Placeholder | Description
    ------------|-------------
    `<VERSION>` | The version number of the output tarball.
    `<RANCHER.YOURDOMAIN.COM>` | The DNS name you pointed at your load balancer.
    `<REGISTRY.YOURDOMAIN.COM:PORT>` | The DNS name for your private registry.
    `<CERTMANAGER_VERSION>` | Cert-manager version running on k8s cluster.

     ```plain
    helm template ./rancher-<VERSION>.tgz --output-dir . \
     --name rancher \
     --namespace cattle-system \
     --set hostname=<RANCHER.YOURDOMAIN.COM> \
     --set certmanager.version=<CERTMANAGER_VERSION> \
     --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher \
     --set systemDefaultRegistry=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Available as of v2.2.0, set a default private registry to be used in Rancher
     --set useBundledSystemChart=true # Available as of v2.3.0, use the packaged Rancher system charts

````

 /accordion 

 accordion id="secret" label="Option B: Certificates From Files using Kubernetes Secrets" 

Create Kubernetes secrets from your own certificates for Rancher to use. The common name for the cert will need to match the `hostname` option in the command below, or the ingress controller will fail to provision the site for Rancher.

Render the Rancher template, declaring your chosen options. Use the reference table below to replace each placeholder. Rancher needs to be configured to use the private registry in order to provision any Rancher launched Kubernetes clusters or Rancher tools.

If you are using a Private CA signed cert, add `--set privateCA=true` following `--set ingress.tls.source=secret`.

| Placeholder                      | Description                                     |
| -------------------------------- | ----------------------------------------------- |
| `<VERSION>`                      | The version number of the output tarball.       |
| `<RANCHER.YOURDOMAIN.COM>`       | The DNS name you pointed at your load balancer. |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | The DNS name for your private registry.         |
```plain
   helm template ./rancher-<VERSION>.tgz --output-dir . \
    --name rancher \
    --namespace cattle-system \
    --set hostname=<RANCHER.YOURDOMAIN.COM> \
    --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher \
    --set ingress.tls.source=secret \
    --set systemDefaultRegistry=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Available as of v2.2.0, set a default private registry to be used in Rancher
    --set useBundledSystemChart=true # Available as of v2.3.0, use the packaged Rancher system charts
````

然后参考 [添加 TLS Secrets](/docs/installation/options/tls-secrets/) 发布证书文件，以便 Rancher 和 Ingress controller 可以使用它们.

 /accordion 

#### D. 安装 Rancher

复制 rendered manifest 文件目录到有权访问 Rancher server 的集群里以完成安装。

使用 `kubectl` 创建 namespaces 和 apply rendered manifests。

如果选择在 [B. 选择 SSL 配置](#b-choose-your-ssl-configuration), 安装 cert-manager。

 accordion id="install-cert-manager" label="Self-Signed Certificate Installs - Install Cert-manager" 

如果您使用的是自签名证书，请安装 cert-manager：

1. Create the namespace for cert-manager.

```plain
kubectl create namespace cert-manager
```

1. Create the cert-manager CustomResourceDefinitions (CRDs).

```plain
kubectl apply -f cert-manager/cert-manager-crd.yaml
```

> **重要提示:**
> 如果您正在运行Kubernetes v1.15或更低版本，则需要在上方的kubectl apply命令中添加`--validate = false标志，否则将收到与 cert-manager 的 CustomResourceDefinition 资源中的 x-kubernetes-preserve-unknown-fields 字段相关的验证错误。这是一个 benign error，由于 kubectl 执行资源验证的方式而发生。

1. 安装 cert-manager。

```plain
kubectl apply -R -f ./cert-manager
```

 /accordion 

Install Rancher:

```plain
kubectl create namespace cattle-system
kubectl -n cattle-system apply -R -f ./rancher
```

**结果:** 如果您正在安装的版本是 Rancher v2.3.0+ ，那么安装已经完成。

#### E. For Rancher versions prior to v2.3.0, Configure System Charts

如果您安装 Rancher 版本是v2.3.0之前,，则将无法使用 packaged system charts. 由于 Rancher system charts 托管在Github中，在离线环境下无法访问 charts。因此，[您必须配置 Rancher system charts](/docs/installation/options/local-system-charts/#setting-up-system-charts-for-rancher-prior-to-v2-3-0)。

#### 额外资源

在安装 Rancher 时，这些资源可能会有帮助:

- [Rancher Helm chart options](/docs/installation/options/chart-options/)
- [Adding TLS secrets](/docs/installation/options/tls-secrets/)
- [Troubleshooting Rancher Kubernetes Installations](/docs/installation/options/troubleshooting/)

 /tab 
 tab "Docker Install" 

Docker 安装是为想要测试 Rancher 的 Rancher 用户准备的。 您不需要在 Kubernetes 集群上运行，而是使用 docker run 命令在单个节点上安装 Rancher server 组件。 由于只有一个节点和一个 Docker 容器，如果该节点宕机，其他节点上就没有可用的 etcd 数据副本，您将丢失 Rancher 服务器的所有数据. **重要提示: 如果您按照 Docker 安装指南安装 Rancher，则没有升级路径将 Docker 安装方式转换为 Kubernetes 安装方式.** 与运行单节点安装不同，您可以选择遵循 Kubernetes 安装指南，但只使用一个节点来安装 Rancher。 然后，您可以扩展您的 Kubernetes 集群中的 etcd 节点，使其成为 Kubernetes 安装。

为了安全起见，在使用 Rancher 时需要 SSL (传输层安全协议)。 SSL 保护所有 Rancher 网络通信，比如当您登录或与集群交互时。

| Environment Variable Key         | Environment Variable Value       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| -------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `CATTLE_SYSTEM_DEFAULT_REGISTRY` | `<REGISTRY.YOURDOMAIN.COM:PORT>` | Configure Rancher server to always pull from your private registry when provisioning clusters.                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `CATTLE_SYSTEM_CATALOG`          | `bundled`                        | Configure Rancher server to use the packaged copy of Helm system charts. The [system charts](https://github.com/rancher/system-charts) repository contains all the catalog items required for features such as monitoring, logging, alerting and global DNS. These [Helm charts](https://github.com/rancher/system-charts) are located in GitHub, but since you are in an air gapped environment, using the charts that are bundled within Rancher is much easier than setting up a Git mirror. _Available as of v2.3.0_ |

> **Do you want to...**
>
> - Configure custom CA root certificate to access your services? See [Custom CA root certificate](/docs/installation/options/chart-options/#additional-trusted-cas).
> - Record all transactions with the Rancher API? See [API Auditing](/docs/installation/other-installation-methods/single-node-docker/#api-audit-log).

- 对于v2.3.0之前的Rancher，您需要将 system-charts 的镜像 repository 放到本地以便Rancher访问。然后，在安装Rancher之后，您将需要配置Rancher 以使用该 repository。有关详细信息，请参阅有关[在v2.3.0之前为Rancher设置 system charts for Rancher](/docs/installation/options/local-system-charts/#setting-up-system-charts-for-rancher-prior-to-v2-3-0)。

从以下选项中选择：

 accordion id="option-a" label="Option A-Default Self-Signed Certificate" 

如果您正在开发或测试环境中安装 Rancher，其中不需要考虑身份验证，请使用 Rancher 生成的自签名证书安装 Rancher。 这个安装选项省去了自己生成证书的麻烦。

登录到您的 Linux 主机，然后运行下面的安装命令。 输入命令时，使用下面的表替换每个占位符。

| Placeholder                      | Description                                                                                                 |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | Your private registry URL and port.                                                                         |
| `<RANCHER_VERSION_TAG>`          | The release tag of the [Rancher version](/docs/installation/options/server-tags/) that you want to install. |

```
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    -e CATTLE_SYSTEM_DEFAULT_REGISTRY=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Set a default private registry to be used in Rancher
    -e CATTLE_SYSTEM_CATALOG=bundled \ #Available as of v2.3.0, use the packaged Rancher system charts
    <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
```

 /accordion 
 accordion id="option-b" label="Option B-Bring Your Own Certificate: Self-Signed" 

在开发或测试环境中，您的团队将访问 Rancher server，创建一个自签名证书用于您的安装，以便您的团队可以验证他们正在连接到  Rancher 实例。

> **先决条件:**
> 在可以联网的主机上使用 OpenSSL 或您选择的其他方法创建自签名证书。
>
> - The certificate files must be in [PEM format](/docs/installation/other-installation-methods/single-node-docker/#pem).
> - In your certificate file, include all intermediate certificates in the chain. Order your certificates with your certificate first, followed by the intermediates. For an example, see [SSL FAQ / Troubleshooting](/docs/installation/other-installation-methods/single-node-docker/#cert-order).

创建证书后，登录到 Linux 主机，然后运行下面的安装命令。 输入命令时，使用下面的表替换每个占位符。 使用 -v 并提供证书的路径，以便将证书挂载到容器中。

| Placeholder                      | Description                                                                                                 |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `<CERT_DIRECTORY>`               | The path to the directory containing your certificate files.                                                |
| `<FULL_CHAIN.pem>`               | The path to your full certificate chain.                                                                    |
| `<PRIVATE_KEY.pem>`              | The path to the private key for your certificate.                                                           |
| `<CA_CERTS>`                     | The path to the certificate authority's certificate.                                                        |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | Your private registry URL and port.                                                                         |
| `<RANCHER_VERSION_TAG>`          | The release tag of the [Rancher version](/docs/installation/options/server-tags/) that you want to install. |

```
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    -v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
    -v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
    -v /<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \
    -e CATTLE_SYSTEM_DEFAULT_REGISTRY=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Set a default private registry to be used in Rancher
    -e CATTLE_SYSTEM_CATALOG=bundled \ #Available as of v2.3.0, use the packaged Rancher system charts
    <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
```

 /accordion 
 accordion id="option-c" label="Option C-Bring Your Own Certificate: Signed by Recognized CA" 

在开发或测试环境中，如果您要公开一个应用程序，请使用由认可核证机关签署的证书，这样您的用户就不会遇到安全警告。

> **先决条件:** 证书文件必须是[PEM 格式](/docs/installation/other-installation-methods/single-node-docker/#pem)。

获得证书后，登录到 Linux 主机，然后运行下面的安装命令。 输入命令时，使用下面的表替换每个占位符。 由于您使用的是由认可核证机关签署的证书CA，因此不必再加载其他 CA 证书文件。

| Placeholder                      | Description                                                                                                 |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `<CERT_DIRECTORY>`               | The path to the directory containing your certificate files.                                                |
| `<FULL_CHAIN.pem>`               | The path to your full certificate chain.                                                                    |
| `<PRIVATE_KEY.pem>`              | The path to the private key for your certificate.                                                           |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | Your private registry URL and port.                                                                         |
| `<RANCHER_VERSION_TAG>`          | The release tag of the [Rancher version](/docs/installation/options/server-tags/) that you want to install. |

> **注意:**  使用 `--no-cacerts` 作为容器的参数来禁用 Rancher 生成的默认 CA 证书。

```
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    --no-cacerts \
    -v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
    -v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
    -e CATTLE_SYSTEM_DEFAULT_REGISTRY=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Set a default private registry to be used in Rancher
    -e CATTLE_SYSTEM_CATALOG=bundled \ #Available as of v2.3.0, use the packaged Rancher system charts
    <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
```

 /accordion 

如果您正在安装 Rancher v2.3.0+ ，那么安装已经完成。

如果您安装 Rancher 版本是v2.3.0之前，则将无法使用 packaged system charts. 由于 Rancher system charts 托管在Github中，在离线环境下无法访问 charts。 因此，[您必须配置 Rancher system charts](/docs/installation/options/local-system-charts/#setting-up-system-charts-for-rancher-prior-to-v2-3-0)。

 /tab 
 /tabs 
