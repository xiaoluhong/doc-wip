---
title: 4. Install Rancher
---
4. 安装Rancher

This section is about how to deploy Rancher for your air gapped environment. An air gapped environment could be where Rancher server will be installed offline, behind a firewall, or behind a proxy. There are _tabs_ for either a high availability (recommended) or a Docker installation.

本节介绍如何为私有环境部署Rancher。 Rancher server将会被离线安装，因为网络可能处于防火墙之后或在代理之后。 _选项卡_用于表述高可用安装（推荐）或Docker安装。

> **Note:** These installation instructions assume you are using Helm 3. For migration of installs started with Helm 2, refer to the official [Helm 2 to 3 migration docs.](https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/) This [section](/docs/installation/options/air-gap-helm2) provides a copy of the older air gap installation instructions for Rancher installed on Kubernetes with Helm 2, and it is intended to be used if upgrading to Helm 3 is not feasible.

> **注意：**这些安装说明假定您使用的是Helm3。有关从Helm 2开始的安装迁移，请参考官方的[Helm2到3迁移文档。](https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/)本节(/docs/installation/options/air-gap-helm2提供了较旧的私有安装说明版本，该说明适用于在装有Helm2的Kubernetes上安装的Rancher。如果无法升级到Helm3，则可以使用它。

 tabs 
 tab "Kubernetes Install (Recommended)" 
 tag "基于Kubernetes安装（推荐）"

Rancher recommends installing Rancher on a Kubernetes cluster. A highly available Kubernetes install is comprised of three nodes running the Rancher server components on a Kubernetes cluster. The persistence layer (etcd) is also replicated on these three nodes, providing redundancy and data duplication in case one of the nodes fails.

Rancher建议在Kubernetes集群上安装Rancher。 高可用的Kubernetes安装包含三个节点。 持久层（etcd）也可以在这三个节点上复制，以便在节点之一发生故障时提供冗余和数据复制。

This section describes installing Rancher in five parts:

- [A. Add the Helm Chart Repository](#a-add-the-helm-chart-repository)
- [B. Choose your SSL Configuration](#b-choose-your-ssl-configuration)
- [C. Render the Rancher Helm Template](#c-render-the-rancher-helm-template)
- [D. Install Rancher](#d-install-rancher)
- [E. For Rancher versions prior to v2.3.0, Configure System Charts](#e-for-rancher-versions-prior-to-v2-3-0-configure-system-charts)

本节分五个部分介绍如何安装Rancher：

- [A. 添加Helm Chart仓库](#a-add-the-helm-chart-repository)
- [B. SSL配置](#b-choose-your-ssl-configuration)
- [C. 配置Rancher Helm 模板](#c-render-the-rancher-helm-template)
- [D. 安装Rancher](#d-install-rancher)
- [E. 针对Rancher2.3.0之前版本配置system-chart](#e-for-rancher-versions-prior-to-v2-3-0-configure-system-charts)

#### A. Add the Helm Chart Repository
添加Helm Chart仓库

From a system that has access to the internet, fetch the latest Helm chart and copy the resulting manifests to a system that has access to the Rancher server cluster.

在Kubernetes集群中添加对应的Rancher仓库

1. If you haven't already, initialize `helm` locally on a workstation that has internet access. Note: Refer to the [Helm version requirements](/docs/installation/options/helm-version) to choose a version of Helm to install Rancher.

   ```plain
   helm init -c
   ```

1. 确保helm已经初始化，如果还没有做，请参考相应的文档[Helm version requirements](/docs/installation/options/helm-version)

   ```plain
   helm init -c
   ```

2. Use `helm repo add` command to add the Helm chart repository that contains charts to install Rancher. For more information about the repository choices and which is best for your use case, see [Choosing a Version of Rancher](/docs/installation/options/server-tags/#helm-chart-repositories).
   {{< release-channel >}}

   ```
   helm repo add rancher-<CHART_REPO> https://releases.rancher.com/server-charts/<CHART_REPO>
   ```

2. 使用`helm repo add`来添加仓库，不同的地址适应不同的Rancher版本，请参考[Choosing a Version of Rancher](/docs/installation/options/server-tags/#helm-chart-repositories).
   {{< release-channel >}}

   ```
   helm repo add rancher-<CHART_REPO> https://releases.rancher.com/server-charts/<CHART_REPO>
   ```

3. Fetch the latest Rancher chart. This will pull down the chart and save it in the current directory as a `.tgz` file.

```plain
helm fetch rancher-<CHART_REPO>/rancher
```

3. 获取最新的Rancher chart，你会看到对应的tgz文件下载到本地

```plain
helm fetch rancher-<CHART_REPO>/rancher
```

> Want additional options? Need help troubleshooting? See [Kubernetes Install: Advanced Options](/docs/installation/k8s-install/helm-rancher/#advanced-configurations).

>是否需要其他选项？ 需要帮助进行故障排除吗？ 请参阅[Kubernetes安装：高级选项](/docs/installation/k8s-install/helm-rancher/#advanced-configurations)。


#### B. Choose your SSL Configuration
SSL配置

Rancher Server is designed to be secure by default and requires SSL/TLS configuration.

Rancher Server在默认情况下被设计为安全的，并且需要SSL/TLS配置。

When Rancher is installed on an air gapped Kubernetes cluster, there are two recommended options for the source of the certificate.

当在私有环境的Kubernetes中安装Rancher时，推荐两种证书生成方式。

> **Note:** If you want terminate SSL/TLS externally, see [TLS termination on an External Load Balancer](/docs/installation/options/chart-options/#external-tls-termination).

**注意：**如果要在外部终止SSL/TLS，请参阅[在外部负载均衡器上终止TLS](/docs/installation/options/chart-options/#external-tls-termination)。

| Configuration                              | Chart option                 | Description                                                                                                                                                   | Requires cert-manager |
| ------------------------------------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| Rancher Generated Self-Signed Certificates | `ingress.tls.source=rancher` | Use certificates issued by Rancher's generated CA (self signed)<br /> This is the **default** and does not need to be added when rendering the Helm template. | yes                   |
| Certificates from Files                    | `ingress.tls.source=secret`  | Use your own certificate files by creating Kubernetes Secret(s). <br /> This option must be passed when rendering the Rancher Helm template.                  | no                    |

| Configuration                              | Chart option                 | Description                                                                                                                                                   | Requires cert-manager |
| ------------------------------------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| Rancher生成的自签名证书                    | `ingress.tls.source=rancher` | 这是默认选项                                                                                                                                                  | yes                   |
| 已有的自签名证书                           | `ingress.tls.source=secret`  |  通过创建Kubernetes Secret（s）使用您自己的证书文件。 <br />渲染Rancher Helm模板时必须传递此选项。                                                            | no                    |

#### C. Render the Rancher Helm Template
配置Rancher Helm模板

When setting up the Rancher Helm template, there are several options in the Helm chart that are designed specifically for air gap installations.

设置Rancher Helm模板时，chart中有几个选项是专门为私有安装设计的。

| Chart Option            | Chart Value                      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ----------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `certmanager.version`   | {`<version>`}                    | Configure proper Rancher TLS issuer depending of running cert-manager version.                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `systemDefaultRegistry` | `<REGISTRY.YOURDOMAIN.COM:PORT>` | Configure Rancher server to always pull from your private registry when provisioning clusters.                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `useBundledSystemChart` | `true`                           | Configure Rancher server to use the packaged copy of Helm system charts. The [system charts](https://github.com/rancher/system-charts) repository contains all the catalog items required for features such as monitoring, logging, alerting and global DNS. These [Helm charts](https://github.com/rancher/system-charts) are located in GitHub, but since you are in an air gapped environment, using the charts that are bundled within Rancher is much easier than setting up a Git mirror. _Available as of v2.3.0_ |

| Chart Option            | Chart Value                      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ----------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `certmanager.version`   | {`<version>`}                    | 根据运行的cert-manager版本配置适当的Rancher TLS颁发者。                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `systemDefaultRegistry` | `<REGISTRY.YOURDOMAIN.COM:PORT>` | 当创建集群时，Rancher server始终从私有镜像仓库中拉取镜像 |
| `useBundledSystemChart` | `true`                           | 配置Rancher server以内置的system-chart， [system-chart](https://github.com/rancher/system-charts)中包含监控，日志，告警和全局DNS等功能所需的Chart。 这些[Helm charts](https://github.com/rancher/system-charts)位于GitHub中，但是由于您处于私有环境中，因此使用Rancher中内置的chart比设置一个Git mirror简单得多。_自v2.3.0起可用_ |

Based on the choice your made in [B. Choose your SSL Configuration](#b-choose-your-ssl-configuration), complete one of the procedures below.

根据您在[B. SSL配置](#b-choose-your-ssl-configuration)做出的选择，完成以下步骤之一。

 accordion id="self-signed" label="选项A - 默认自签名方式" 

By default, Rancher generates a CA and uses cert-manager to issue the certificate for access to the Rancher server interface.

默认情况下，Rancher会生成一个CA并使用cert-manager颁发证书以访问Rancher server界面。

> **Note:**
> Recent changes to cert-manager require an upgrade. If you are upgrading Rancher and using a version of cert-manager older than v0.11.0, please see our [upgrade cert-manager documentation](/docs/installation/options/upgrading-cert-manager/).

> **注意：**>最近对cert-manager进行的更改需要升级。 如果您要升级Rancher并使用版本低于v0.11.0的cert-manager，请参阅我们的[升级cert-manager文档](/docs/installation/options/upgradeing-cert-manager/)。

1. From a system connected to the internet, add the cert-manager repo to Helm.

   ```plain
   helm repo add jetstack https://charts.jetstack.io
   helm repo update
   ```

1. 联网添加cert-manager仓库

   ```plain
   helm repo add jetstack https://charts.jetstack.io
   helm repo update
   ```

1. Fetch the latest cert-manager chart available from the [Helm chart repository](https://hub.helm.sh/charts/jetstack/cert-manager).

   ```plain
   helm fetch jetstack/cert-manager --version v0.12.0
   ```

1. 从[Helm chart repository]（https://hub.helm.sh/charts/jetstack/cert-manager）中获取最新的cert-manager chart。

   ```plain
   helm fetch jetstack/cert-manager --version v0.12.0
   ```

1. Render the cert manager template with the options you would like to use to install the chart. Remember to set the `image.repository` option to pull the image from your private registry. This will create a `cert-manager` directory with the Kubernetes manifest files.

   ```plain
   helm template cert-manager ./cert-manager-v0.12.0.tgz --output-dir . \
       --namespace cert-manager \
       --set image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-controller \
       --set webhook.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-webhook \
       --set cainjector.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-cainjector
   ```

1. 配置chart 模板参数， 切记设置`image.repository`以便从私有镜像仓库中拉取图像。
   ```plain
   helm template cert-manager ./cert-manager-v0.12.0.tgz --output-dir . \
       --namespace cert-manager \
       --set image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-controller \
       --set webhook.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-webhook \
       --set cainjector.image.repository=<REGISTRY.YOURDOMAIN.COM:PORT>/quay.io/jetstack/cert-manager-cainjector
   ```


1. Download the required CRD file for cert-manager
1. 下载cert-manager所需的CRD文件
   ```plain
   curl -L -o cert-manager/cert-manager-crd.yaml https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml
   ```
1. Render the Rancher template, declaring your chosen options. Use the reference table below to replace each placeholder. Rancher needs to be configured to use the private registry in order to provision any Rancher launched Kubernetes clusters or Rancher tools.
1. 配置Rancher模板，参考下表中的关键选项。

    Placeholder | Description
    ------------|-------------
    `<VERSION>` | The version number of the output tarball.
    `<RANCHER.YOURDOMAIN.COM>` | The DNS name you pointed at your load balancer.
    `<REGISTRY.YOURDOMAIN.COM:PORT>` | The DNS name for your private registry.
    `<CERTMANAGER_VERSION>` | Cert-manager version running on k8s cluster.

    Placeholder | Description
    ------------|-------------
    `<VERSION>` | 对应Rancher版本
    `<RANCHER.YOURDOMAIN.COM>` | 负载均衡对应的DNS 
    `<REGISTRY.YOURDOMAIN.COM:PORT>` | 私有镜像库对应的DNS
    `<CERTMANAGER_VERSION>` | Cert-manager版本

     ```plain
    helm template rancher ./rancher-<VERSION>.tgz --output-dir . \
     --namespace cattle-system \
     --set hostname=<RANCHER.YOURDOMAIN.COM> \
     --set certmanager.version=<CERTMANAGER_VERSION> \
     --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher \
     --set systemDefaultRegistry=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Available as of v2.2.0, set a default private registry to be used in Rancher
     --set useBundledSystemChart=true # Available as of v2.3.0, use the packaged Rancher system charts

````

 /accordion 

 accordion id="secret" label="选项 B: 使用来自Kubernetes Secrets的证书" 

Create Kubernetes secrets from your own certificates for Rancher to use. The common name for the cert will need to match the `hostname` option in the command below, or the ingress controller will fail to provision the site for Rancher.

根据您自己的证书创建Kubernetes secrets，以供Rancher使用。 证书的通用名称将需要与以下命令中的`主机名`选项匹配，否则ingress controller将无法为Rancher设置入口。

Render the Rancher template, declaring your chosen options. Use the reference table below to replace each placeholder. Rancher needs to be configured to use the private registry in order to provision any Rancher launched Kubernetes clusters or Rancher tools.

设置Rancher模板，声明您选择的选项。 使用下面表中的参考选项，需要给Rancher配置使用私有镜像库。

If you are using a Private CA signed cert, add `--set privateCA=true` following `--set ingress.tls.source=secret`.

如果您使用的是由私有CA签名的证书，则在--set ingress.tls.source=secret`之后添加`--set privateCA=true`。

| Placeholder                      | Description                                     |
| -------------------------------- | ----------------------------------------------- |
| `<VERSION>`                      | The version number of the output tarball.       |
| `<RANCHER.YOURDOMAIN.COM>`       | The DNS name you pointed at your load balancer. |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | The DNS name for your private registry.         |

| Placeholder                      | Description                                     |
| -------------------------------- | ----------------------------------------------- |
| `<VERSION>`                      | Rancher版本
| `<RANCHER.YOURDOMAIN.COM>`       | 负载均衡器对应的DNS |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | 私有镜像库对应的DNS |

```plain
   helm template rancher ./rancher-<VERSION>.tgz --output-dir . \
    --namespace cattle-system \
    --set hostname=<RANCHER.YOURDOMAIN.COM> \
    --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher \
    --set ingress.tls.source=secret \
    --set systemDefaultRegistry=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Available as of v2.2.0, set a default private registry to be used in Rancher
    --set useBundledSystemChart=true # Available as of v2.3.0, use the packaged Rancher system charts
````

Then refer to [Adding TLS Secrets](/docs/installation/options/tls-secrets/) to publish the certificate files so Rancher and the ingress controller can use them.

然后请参考[添加TLS Secrets](/docs/installation/options/tls-secrets/)发布证书文件，以便Rancher和ingress controller可以使用它们。

 /accordion 

#### D. Install Rancher
安装Rancher

Copy the rendered manifest directories to a system that has access to the Rancher server cluster to complete installation.
将以上配置完毕的内容准备妥当，完成最后的安装。

Use `kubectl` to create namespaces and apply the rendered manifests.

使用`kubectl`创建名称空间并安装配置好的chart。

If you chose to use self-signed certificates in [B. Choose your SSL Configuration](#b-choose-your-ssl-configuration), install cert-manager.

如果您选择自签名证书在[B. SSL配置](#b-choose-your-ssl-configuration)，则安装cert-manager。

 accordion id="install-cert-manager" label="自签名证书 - 安装Cert-manager" 

If you are using self-signed certificates, install cert-manager:

如果您使用的是自签名证书，请安装cert-manager：

1. Create the namespace for cert-manager.

1. 为cert-manager创建namespace。

```plain
kubectl create namespace cert-manager
```

1. Create the cert-manager CustomResourceDefinitions (CRDs).

1. 创建cert-manager CRD

```plain
kubectl apply -f cert-manager/cert-manager-crd.yaml
```

    > **Note:**
    > If you are running Kubernetes v1.15 or below, you will need to add the `--validate=false` flag to your `kubectl apply` command above, or else you will receive a validation error relating to the `x-kubernetes-preserve-unknown-fields` field in cert-manager’s CustomResourceDefinition resources. This is a benign error and occurs due to the way kubectl performs resource validation.

1. Launch cert-manager.

1. 启动cert-manager.

```plain
kubectl apply -R -f ./cert-manager
```

 /accordion 

Install Rancher:

安装Rancher：

```plain
kubectl create namespace cattle-system
kubectl -n cattle-system apply -R -f ./rancher
```

**Step Result:** If you are installing Rancher v2.3.0+, the installation is complete.

**步骤结果：**如果要安装Rancher v2.3.0 +，则安装完成。

#### E. For Rancher versions prior to v2.3.0, Configure System Charts
E.对于v2.3.0之前的Rancher版本，配置system-charts

If you are installing Rancher versions prior to v2.3.0, you will not be able to use the packaged system charts. Since the Rancher system charts are hosted in Github, an air gapped installation will not be able to access these charts. Therefore, you must [configure the Rancher system charts](/docs/installation/options/local-system-charts/#setting-up-system-charts-for-rancher-prior-to-v2-3-0).
如果要安装v2.3.0之前的Rancher版本，则将无法使用内置打包的system-charts。 由于Rancher system-charts托管在Github中，因此，私有安装将无法访问charts。 因此，您必须[配置Rancher system-charts](/docs/installation/options/local-system-charts/#setting-up-system-charts-for-rancher-prior-to-v2-3-0)。

#### Additional Resources
其他资源

These resources could be helpful when installing Rancher:

- [Rancher Helm chart options](/docs/installation/options/chart-options/)
- [Adding TLS secrets](/docs/installation/options/tls-secrets/)
- [Troubleshooting Rancher Kubernetes Installations](/docs/installation/options/troubleshooting/)

这些资源在安装Rancher时可能会有所帮助：

- [Rancher Helm chart选项](/docs/installation/options/chart-options/)
- [添加TLS secrets](/docs/installation/options/tls-secrets/)
- [安装过程的故障排除](/docs/installation/options/troubleshooting/)

 /tab 
 tab "Docker Install" 

The Docker installation is for Rancher users that are wanting to **test** out Rancher. Instead of running on a Kubernetes cluster, you install the Rancher server component on a single node using a `docker run` command. Since there is only one node and a single Docker container, if the node goes down, there is no copy of the etcd data available on other nodes and you will lose all the data of your Rancher server. **Important: If you install Rancher following the Docker installation guide, there is no upgrade path to transition your Docker installation to a Kubernetes Installation.** Instead of running the single node installation, you have the option to follow the Kubernetes Install guide, but only use one node to install Rancher. Afterwards, you can scale up the etcd nodes in your Kubernetes cluster to make it a Kubernetes Installation.
Docker安装适用于想要对Rancher进行测试的Rancher用户。 您可以使用docker run命令在单个节点上安装Rancher服务器组件，而不是在Kubernetes集群上运行。 由于只有一个节点和一个Docker容器，因此，如果该节点发生故障，则其他节点上没有可用的etcd数据副本，您将丢失Rancher服务器的所有数据。 **重要提示：如果您按照Docker安装指南安装Rancher，则没有升级路径可将Docker安装过渡到Kubernetes安装。**除了运行单节点安装，您还可以选择遵循Kubernetes安装指南， 但只能使用一个节点来安装Rancher。 之后，您可以扩展Kubernetes集群中的etcd节点，使其成为Kubernetes安装。

For security purposes, SSL (Secure Sockets Layer) is required when using Rancher. SSL secures all Rancher network communication, like when you login or interact with a cluster.
为了安全起见，使用Rancher时需要SSL。 SSL保护所有Rancher网络通信的安全，例如在您登录集群或与集群交互时。

| Environment Variable Key         | Environment Variable Value       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| -------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `CATTLE_SYSTEM_DEFAULT_REGISTRY` | `<REGISTRY.YOURDOMAIN.COM:PORT>` | Configure Rancher server to always pull from your private registry when provisioning clusters.                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `CATTLE_SYSTEM_CATALOG`          | `bundled`                        | Configure Rancher server to use the packaged copy of Helm system charts. The [system charts](https://github.com/rancher/system-charts) repository contains all the catalog items required for features such as monitoring, logging, alerting and global DNS. These [Helm charts](https://github.com/rancher/system-charts) are located in GitHub, but since you are in an air gapped environment, using the charts that are bundled within Rancher is much easier than setting up a Git mirror. _Available as of v2.3.0_ |

| Environment Variable Key         | Environment Variable Value       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| -------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `CATTLE_SYSTEM_DEFAULT_REGISTRY` | `<REGISTRY.YOURDOMAIN.COM:PORT>` |  在配置集群时，将Rancher server配置为始终从您的私有镜像库中拉取镜像。  |
| `CATTLE_SYSTEM_CATALOG`          | `bundled`                        | 配置Rancher server以内置的system-chart， [system-chart](https://github.com/rancher/system-charts)中包含监控，日志，告警和全局DNS等功能所需的Chart。 这些[Helm charts](https://github.com/rancher/system-charts)位于GitHub中，但是由于您处于私有环境中，因此使用Rancher中内置的chart比设置一个Git mirror简单得多。_自v2.3.0起可用_ |

> **Do you want to...**
>
> - Configure custom CA root certificate to access your services? See [Custom CA root certificate](/docs/installation/options/custom-ca-root-certificate/).
> - Record all transactions with the Rancher API? See [API Auditing](/docs/installation/other-installation-methods/single-node-docker/#api-audit-log).

> - 配置自定义CA根证书以访问您的服务？ 请参阅[自定义CA根证书](/docs/installation/options/custom-ca-root-certificate/).
> - 开启API审计日志，请参阅[API审计](/docs/installation/other-installation-methods/single-node-docker/#api-audit-log).

- For Rancher prior to v2.3.0, you will need to mirror the `system-charts` repository to a location in your network that Rancher can reach. Then, after Rancher is installed, you will need to configure Rancher to use that repository. For details, refer to the documentation on [setting up the system charts for Rancher prior to v2.3.0.](/docs/installation/options/local-system-charts/#setting-up-system-charts-for-rancher-prior-to-v2-3-0)
- -对于v2.3.0之前的Rancher，您需要设置Git mirror将system-charts置于网络中Rancher可以访问的位置。 然后，在安装Rancher之后，您将需要配置Rancher以使用该Git仓库。 有关详细信息，请参阅[在v2.3.0之前为Rancher设置system-charts。](/docs/installation/options/local-system-charts/#setting-up-system-charts-for-rancher-prior-to-v2-3-0）

Choose from the following options:
选择下面的选项

 accordion id="option-a" label="Option A-Default Self-Signed Certificate" 
 选项A - 默认Rancher自签名证书

If you are installing Rancher in a development or testing environment where identity verification isn't a concern, install Rancher using the self-signed certificate that it generates. This installation option omits the hassle of generating a certificate yourself.

如果要在不涉及身份验证的开发或测试环境中安装Rancher，请使用其生成的自签名证书安装Rancher。 此安装选项省去了自己生成证书的麻烦。

Log into your Linux host, and then run the installation command below. When entering the command, use the table below to replace each placeholder.

登录到Linux主机，然后运行下面的安装命令。 输入命令时，请参考下面的配置。

| Placeholder                      | Description                                                                                                 |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | Your private registry URL and port.                                                                         |
| `<RANCHER_VERSION_TAG>`          | The release tag of the [Rancher version](/docs/installation/options/server-tags/) that you want to install. |

| Placeholder                      | Description                                                                                                 |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | 私有镜像库地址                                                                        |
| `<RANCHER_VERSION_TAG>`          | 您要安装的[Rancher版本](/docs/installation/options/server-tags/) |

```
docker run -d --restart=unless-stopped \
    -p 80:80 -p 443:443 \
    -e CATTLE_SYSTEM_DEFAULT_REGISTRY=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Set a default private registry to be used in Rancher
    -e CATTLE_SYSTEM_CATALOG=bundled \ #Available as of v2.3.0, use the packaged Rancher system charts
    <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
```

 /accordion 
 accordion id="option-b" label="Option B-Bring Your Own Certificate: Self-Signed" 
 选项B -  独立自签名证书

In development or testing environments where your team will access your Rancher server, create a self-signed certificate for use with your install so that your team can verify they're connecting to your instance of Rancher.

在您的团队将访问Rancher服务器的开发或测试环境中，创建一个自签名证书以供您的安装使用，以便您的团队可以验证它们是否正在连接到Rancher实例。

> **Prerequisites:**
> From a computer with an internet connection, create a self-signed certificate using [OpenSSL](https://www.openssl.org/) or another method of your choice.
>
> - The certificate files must be in [PEM format](/docs/installation/other-installation-methods/single-node-docker/#pem).
> - In your certificate file, include all intermediate certificates in the chain. Order your certificates with your certificate first, followed by the intermediates. For an example, see [SSL FAQ / Troubleshooting](/docs/installation/other-installation-methods/single-node-docker/#cert-order).

> **先决条件：**
> 在具有互联网连接的计算机上，使用[OpenSSL](https://www.openssl.org/)或您选择的其他方法创建自签名证书。
>
> - 证书文件必须为[PEM格式](/docs/installation/other-installation-methods/single-node-docker/#pem)。
> - 其他证书问题[SSL FAQ / Troubleshooting](/docs/installation/other-installation-methods/single-node-docker/#cert-order)

After creating your certificate, log into your Linux host, and then run the installation command below. When entering the command, use the table below to replace each placeholder. Use the `-v` flag and provide the path to your certificates to mount them in your container.
创建证书后，登录到Linux主机，然后运行以下安装命令。相关参数参考下表：

| Placeholder                      | Description                                                                                                 |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `<CERT_DIRECTORY>`               | The path to the directory containing your certificate files.                                                |
| `<FULL_CHAIN.pem>`               | The path to your full certificate chain.                                                                    |
| `<PRIVATE_KEY.pem>`              | The path to the private key for your certificate.                                                           |
| `<CA_CERTS>`                     | The path to the certificate authority's certificate.                                                        |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | Your private registry URL and port.                                                                         |
| `<RANCHER_VERSION_TAG>`          | The release tag of the [Rancher version](/docs/installation/options/server-tags/) that you want to install. |

| Placeholder                      | Description                                                                                                 |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `<CERT_DIRECTORY>`               | 证书文件所在目录                                                |
| `<FULL_CHAIN.pem>`               | 证书链文件路径                                                                    |
| `<PRIVATE_KEY.pem>`              | 证书私有秘钥路径                                                           |
| `<CA_CERTS>`                     | 证书颁发机构的证书的路径                                                       |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | 私有镜像库                                                                        |
| `<RANCHER_VERSION_TAG>`          | 您要安装的[Rancher版本](/docs/installation/options/server-tags) |


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
 使用公认CA颁发的证书

In development or testing environments where you're exposing an app publicly, use a certificate signed by a recognized CA so that your user base doesn't encounter security warnings.
在要公开展示应用程序的开发或测试环境中，请使用由公认的CA签名的证书，这样您的用户群就不会遇到安全警告。

> **Prerequisite:** The certificate files must be in [PEM format](/docs/installation/other-installation-methods/single-node-docker/#pem).
> **前提：**证书文件必须为[PEM格式](/docs/installation/other-installation-methods/single-node-docker/#pem）。

After obtaining your certificate, log into your Linux host, and then run the installation command below. When entering the command, use the table below to replace each placeholder. Because your certificate is signed by a recognized CA, mounting an additional CA certificate file is unnecessary.
获得证书后，登录到Linux主机，然后运行下面的安装命令。由于您的证书是由公认的CA签名的，因此不需要安装其他CA证书文件。

| Placeholder                      | Description                                                                                                 |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `<CERT_DIRECTORY>`               | The path to the directory containing your certificate files.                                                |
| `<FULL_CHAIN.pem>`               | The path to your full certificate chain.                                                                    |
| `<PRIVATE_KEY.pem>`              | The path to the private key for your certificate.                                                           |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | Your private registry URL and port.                                                                         |
| `<RANCHER_VERSION_TAG>`          | The release tag of the [Rancher version](/docs/installation/options/server-tags/) that you want to install. |

> **Note:** Use the `--no-cacerts` as argument to the container to disable the default CA certificate generated by Rancher.
> **注意：**使用`--no-cacerts`作为容器的参数来禁用Rancher生成的默认CA证书。

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

If you are installing Rancher v2.3.0+, the installation is complete.
如果要安装Rancher v2.3.0 +，则安装完成。

If you are installing Rancher versions prior to v2.3.0, you will not be able to use the packaged system charts. Since the Rancher system charts are hosted in Github, an air gapped installation will not be able to access these charts. Therefore, you must [configure the Rancher system charts](/docs/installation/options/local-system-charts/#setting-up-system-charts-for-rancher-prior-to-v2-3-0).
如果要安装v2.3.0之前的Rancher版本，则将无法使用内置的system-charts。 由于Rancher system-charts托管在Github中，因此， 私有安装将无法访问这些charts。 因此，您必须[配置Rancher system-charts](/docs/installation/options/local-system-charts/#setting-up-system-charts-for-rancher-prior-to-v2-3-0)。
 /tab 
 /tabs 
