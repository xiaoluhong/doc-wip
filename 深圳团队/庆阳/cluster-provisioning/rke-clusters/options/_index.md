---
标题: 集群配置参考
---

当您配置[使用RKE启动](/docs/cluster-provisioning/rke-clusters/)的新集群时，您可以选择自定义Kubernetes选项。

您可以通过以下两种方式之一配置Kubernetes选项:

- [Rancher UI](#rancher-ui): 使用Rancher UI来选择在设置Kubernetes集群时通常自定义的选项。
- [集群配置文件](#cluster-config-file): 高级用户可以创建RKE配置文件，而不是使用Rancher UI为集群选择Kubernetes选项。 通过使用配置文件，您可以通过在YAML中指定它们来设置RKE安装中可用的任何选项，但system_images配置除外。

在Rancher v2.0.0-v2.2.x，配置文件与[Rancher Kubernetes引擎的集群配置文件]({{<baseurl>}}/rke/latest/en/config-options/)相同，Rancher用于配置集群的工具。 在Rancher v2.3.0中，rke信息仍然包含在配置文件中，但它与其他选项分开，因此RKE集群配置选项嵌套在`rancher_kubernetes_engine_config`指令下。 有关详细信息，请参阅关于[集群配置文件。](#cluster-config-file)

本节是集群配置参考，涵盖以下主题:

- [Rancher UI选项](#rancher-ui-options)
  - [Kubernetes版本](#kubernetes-version)
  - [网络驱动](#network-provider)
  - [Kubernetes云提供商](#kubernetes-cloud-providers)
  - [私有镜像仓库](#private-registries)
  - [授权集群访问地址](#authorized-cluster-endpoint)
- [高级选项](#advanced-options)
  - [NGINX Ingress](#nginx-ingress)
  - [Node port范围](#node-port-range)
  - [监控指标](#metrics-server-monitoring)
  - [Pod安全策略](#pod-security-policy-support)
  - [主机Docker版本](#docker-version-on-nodes)
  - [Docker根目录](#docker-root-directory)
  - [Etcd备份轮换](#recurring-etcd-snapshots)
- [集群配置文件](#cluster-config-file)
  - [Rancher v2.3.0+ 配置文件结构](#config-file-structure-in-rancher-v2-3-0+)
  - [Rancher v2.0.0-v2.2.x 配置文件结构](#config-file-structure-in-rancher-v2-0-0-v2-2-x)
  - [默认DNS插件](#default-dns-provider)
- [Rancher特有参数](#rancher-specific-parameters)

## Rancher UI选项

使用[Rancher Launched Kubernetes](/docs/cluster-provisioning/rke-clusters)中描述的一个选项创建集群时，您可以使用**集群选项**部分配置基本的Kubernetes选项。

#### Kubernetes版本

安装在集群节点上的Kubernetes版本。 Rancher基于[hyperkube](https://github.com/rancher/hyperkube)打包自己的Kubernetes版本。

#### 网络驱动

[网络驱动](https://kubernetes.io/docs/concepts/cluster-administration/networking/)集群使用。 有关不同网络提供商的更多详细信息，请查看我们的[网络常见问题解答](/docs/faq/networking/cni-providers/)。

> **注意:** 启动集群后，您无法更改网络插件。 因此，请谨慎选择要使用的网络插件，因为Kubernetes不允许在网络插件之间切换。 使用网络插件创建集群后，更改网络插件将要求拆除整个集群及其所有应用程序。

开箱即用，Rancher与以下网络插件兼容:

- [Canal](https://github.com/projectcalico/canal)
- [Flannel](https://github.com/coreos/flannel#flannel)
- [Calico](https://docs.projectcalico.org/v3.11/introduction/)
- [Weave](https://github.com/weaveworks/weave) (v2.2.0可用)

**Canal注意事项:**

在v2.0.0-v2.0.4和v2.0.6中，这是这些集群的默认选项是开启网络隔离的Canal。 在自动启用网络隔离后，它阻止了[projects](/docs/k8s-in-rancher/projects-and-namespaces/)之间的任何pod通信。

从v2.0.7开始，如果您使用Canal，您还可以选择使用**网络隔离**，这将启用或禁用不同[projects](/docs/k8s-in-rancher/projects-and-namespaces/)中的pod之间的通信。

> **Rancher v2.0.0 - v2.0.6用户注意**
>
> - 在以前的Rancher版本中，Canal隔离了项目网络通信，但没有禁用它的选项。 如果您正在使用任何这些Rancher版本，请注意，使用Canal可以防止不同项目中的pad之间的所有通信。
> - 如果您有使用Canal的集群并且正在升级到v2.0.7，那么这些集群默认情况下会启用Project Network隔离。 如果要禁用项目网络隔离，请编辑集群并禁用该选项。

**Flannel注意事项:**

在v2.0.5中，这是默认选项，它不会阻止项目之间的任何网络隔离。

**Weave注意事项:**

当Weave被选为网络插件时，Rancher将通过生成随机密码自动启用加密。 如果要手动指定密码，请参阅如何使用[Config文件](/docs/cluster-provisioning/rke-clusters/options/#config-file)和[Weave网络插件选项]({{<baseurl>}}/rke/latest/en/config-options/add-ons/network-plugins/#weave-network-plug-in-options)配置集群。

#### Kubernetes云提供商

您可以配置[Kubernetes云提供商](/docs/cluster-provisioning/rke-clusters/options/cloud-providers). 如果您想要在Kubernetes中使用[卷和存储](/docs/k8s-in-rancher/volumes-and-storage/), 通常，您必须选择特定的云提供商才能使用它。 例如，如果您想使用Amazon EBS，则需要选择`aws`云提供商。

> **注意:** 如果您要使用的云提供商未列为选项，则需要使用[config file选项](#config-file)来配置云提供商。 请参考[RKE云提供商文档]({{<baseurl>}}/rke/latest/en/config-options/cloud-providers/)关于如何配置云提供商。

如果要查看集群的所有配置选项，请单击右下角的**显示高级选项**。 高级选项如下所述:

#### 私有镜像仓库

_v2.2.0可用_

集群级别的私有镜像仓库配置仅用于设置集群。

在Rancher中设置私有镜像仓库的主要方法有两种：通过全局视图中的**设置**选项卡设置[全局默认镜像仓库](/docs/admin-settings/config-private-registry)，以及通过集群级别设置中的高级选项设置私有镜像仓库。 全局默认镜像仓库旨在用于不需要凭据的镜像仓库的air-gapped设置。 集群级别的私有镜像仓库旨在用于所有需要凭据设置的私有镜像仓库。

如果您的私有镜像仓库需要凭据，则需要通过编辑从镜像仓库中提取镜像的每个集群的集群选项将凭据传递给Rancher。

私有镜像仓库配置选项告诉Rancher将在集群中使用的[系统镜像]({{<baseurl>}}/rke/latest/en/config-options/system-images/)或[addon镜像]({{<baseurl>}}/rke/latest/en/config-options/add-ons/)的位置。

- **系统镜像**是维护Kubernetes集群所需的组件。
- **Add-ons**用于部署多个集群组件包括网络插件、ingress控制器、DNS插件或指标服务器。

请参阅[RKE有关私有镜像仓库文档]({{<baseurl>}}/rke/latest/en/config-options/private-registries/)了解私有镜像仓库在集群配置过程中组件的更多信息。

#### 授权集群访问地址

_v2.2.0可用_

授权集群端点可用于直接访问Kubernetes API服务器，无需通过Rancher进行通信。

> 授权的集群端点仅适用于Rancher启动的Kubernetes集群。 换句话说，它只适用于rancher[使用RKE](/docs/overview/architecture/#tools-for-provisioning-kubernetes-clusters)来配置集群的集群。 它不适用于托管的Kubernetes提供商中的集群，例如Amazon的EKS。

这在Rancher启动的Kubernetes集群中默认启用，使用具有`controlplane`角色的节点的IP和默认的Kubernetes自签名证书。

有关授权的集群终结点如何工作以及使用它的原因的更多详细信息，请参阅[体系结构部分。](/docs/overview/architecture/#4-authorized-cluster-endpoint)

我们建议将负载均衡器与授权的集群终结点一起使用。 有关详细信息，请参阅[推荐架构部分。](/docs/overview/architecture-recommendations/#architecture-for-an-authorized-cluster-endpoint)

## 高级选项

在Rancher UI中创建集群时，以下选项可用。 它们位于**高级选项下。**

#### NGINX Ingress

启用或禁用[NGINX ingress控制器]({{<baseurl>}}/rke/latest/en/config-options/add-ons/ingress-controller/)的选项。

#### Node Port范围

更改可用于[节点端口服务](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport)的端口范围的选项默认值为`30000-32767`。

#### 监控指标

启用或禁用[指标服务器]({{<baseurl>}}/rke/latest/en/config-options/add-ons/metrics-server/)的选项。

#### Pod安全策略

选项启用并选择默认的[Pose安全策略](/docs/admin-settings/pod-security-policies)。 必须先配置现有Pod安全策略，然后才能使用此选项。

#### 主机Docker版本

要求在添加到集群的集群节点上安装[受支持的Docker版本](/docs/installation/requirements/)或允许在集群节点上安装不受支持的Docker版本。

#### Docker根目录

如果要添加到集群的节点配置了非默认的Docker根目录（默认为`/var/lib/docker`），请在此选项中指定正确的Docker根目录。

#### Etcd备份轮换

选项来启用或禁用[Etcd备份轮换]({{<baseurl>}}/rke/latest/en/etcd-snapshots/#etcd-recurring-snapshots)。

## 集群配置文件

高级用户可以创建RKE配置文件，而不是使用Rancher UI为集群选择Kubernetes选项。 使用配置文件允许您在RKE安装中设置任何[可用选项]({{<baseurl>}}/rke/latest/en/config-options/)，除了`system_images`配置。 使用Rancher UI或API创建集群时不支持`system_images`选项。

> **注意:** 在Rancher v2.0.5和v2.0.6中，配置文件（YAML）中的服务名称应该只包含下划线`kube_api`和`kube_controller`。

- 要直接从Rancher UI编辑RKE配置文件，请单击**编辑为YAML**。
- 要从现有RKE文件读取请单击**从文件读取**。

![image](/img/rancher/cluster-options-yaml.png)

根据您的Rancher版本，配置文件的结构是不同的。 以下是Rancher v2.0.0-v2.2的示例配置文件。x和牧场主v2.3.0+。

#### Rancher v2.3.0+ 配置文件结构

RKE（Rancher Kubernetes引擎）是Rancher用来配置Kubernetes集群的工具。 Rancher的集群配置文件过去与[RKE config files,]({{<baseurl>}}/rke/latest/en/example-yamls/)具有相同的结构，但结构发生了变化，因此在Rancher中，RKE集群配置项目与非RKE配置项目分开。 因此，集群的配置需要嵌套在集群配置文件中的'rancher_kubernetes_engine_config'指令下。 使用早期版本的Rancher创建的集群配置文件将需要更新此格式。 下面包含一个示例集群配置文件。

 accordion id="v2.3.0-cluster-config-file" label="Example Cluster Config File for Rancher v2.3.0+" 

```yaml
#
## Cluster Config
#
docker_root_dir: /var/lib/docker
enable_cluster_alerting: false
enable_cluster_monitoring: false
enable_network_policy: false
local_cluster_auth_endpoint:
  enabled: true
#
## Rancher Config
#
rancher_kubernetes_engine_config: # Your RKE template config goes here.
  addon_job_timeout: 30
  authentication:
    strategy: x509
  ignore_docker_version: true
  #
  ## # Currently only nginx ingress provider is supported.
  ## # To disable ingress controller, set `provider: none`
  ## # To enable ingress on specific nodes, use the node_selector, eg:
  ##    provider: nginx
  ##    node_selector:
  ##      app: ingress
  #
  ingress:
    provider: nginx
  kubernetes_version: v1.15.3-rancher3-1
  monitoring:
    provider: metrics-server
  #
  ##   If you are using calico on AWS
  #
  ##    network:
  ##      plugin: calico
  ##      calico_network_provider:
  ##        cloud_provider: aws
  #
  ## # To specify flannel interface
  #
  ##    network:
  ##      plugin: flannel
  ##      flannel_network_provider:
  ##      iface: eth1
  #
  ## # To specify flannel interface for canal plugin
  #
  ##    network:
  ##      plugin: canal
  ##      canal_network_provider:
  ##        iface: eth1
  #
  network:
    options:
      flannel_backend_type: vxlan
    plugin: canal
  #
  ##    services:
  ##      kube-api:
  ##        service_cluster_ip_range: 10.43.0.0/16
  ##      kube-controller:
  ##        cluster_cidr: 10.42.0.0/16
  ##        service_cluster_ip_range: 10.43.0.0/16
  ##      kubelet:
  ##        cluster_domain: cluster.local
  ##        cluster_dns_server: 10.43.0.10
  #
  services:
    etcd:
      backup_config:
        enabled: true
        interval_hours: 12
        retention: 6
        safe_timestamp: false
      creation: 12h
      extra_args:
        election-timeout: 5000
        heartbeat-interval: 500
      gid: 0
      retention: 72h
      snapshot: false
      uid: 0
    kube_api:
      always_pull_images: false
      pod_security_policy: false
      service_node_port_range: 30000-32767
  ssh_agent_auth: false
windows_prefered_cluster: false
```

 /accordion 

#### Rancher v2.0.0-v2.2.x 配置文件结构

下面包含一个示例集群配置文件。

 accordion id="prior-to-v2.3.0-cluster-config-file" label="Example Cluster Config File for Rancher v2.0.0-v2.2.x" 

```yaml
addon_job_timeout: 30
authentication:
  strategy: x509
ignore_docker_version: true
#
## # Currently only nginx ingress provider is supported.
## # To disable ingress controller, set `provider: none`
## # To enable ingress on specific nodes, use the node_selector, eg:
##    provider: nginx
##    node_selector:
##      app: ingress
#
ingress:
  provider: nginx
kubernetes_version: v1.15.3-rancher3-1
monitoring:
  provider: metrics-server
#
##   If you are using calico on AWS
#
##    network:
##      plugin: calico
##      calico_network_provider:
##        cloud_provider: aws
#
## # To specify flannel interface
#
##    network:
##      plugin: flannel
##      flannel_network_provider:
##      iface: eth1
#
## # To specify flannel interface for canal plugin
#
##    network:
##      plugin: canal
##      canal_network_provider:
##        iface: eth1
#
network:
  options:
    flannel_backend_type: vxlan
  plugin: canal
#
##    services:
##      kube-api:
##        service_cluster_ip_range: 10.43.0.0/16
##      kube-controller:
##        cluster_cidr: 10.42.0.0/16
##        service_cluster_ip_range: 10.43.0.0/16
##      kubelet:
##        cluster_domain: cluster.local
##        cluster_dns_server: 10.43.0.10
#
services:
  etcd:
    backup_config:
      enabled: true
      interval_hours: 12
      retention: 6
      safe_timestamp: false
    creation: 12h
    extra_args:
      election-timeout: 5000
      heartbeat-interval: 500
    gid: 0
    retention: 72h
    snapshot: false
    uid: 0
  kube_api:
    always_pull_images: false
    pod_security_policy: false
    service_node_port_range: 30000-32767
ssh_agent_auth: false
```

 /accordion 

#### 默认DNS插件

下表显示了默认情况下部署的DNS提供商。 有关如何配置不同的DNS提供程序的详细信息，请参阅[关于DNS提供程序的文档]({{<baseurl>}}/rke/latest/en/config-options/add-ons/dns/)。 Coredn只能在Kubernetes v1.12.0及更高版本上使用。

| Rancher版本   | Kubernetes版本 | 默认DNS插件 |
| ----------------- | ------------------ | -------------------- |
| v2.2.5 and higher | v1.14.0 and higher | CoreDNS              |
| v2.2.5 and higher | v1.13.x and lower  | kube-dns             |
| v2.2.4 and lower  | any                | kube-dns             |

## Rancher特有参数

_v2.2.0可用_

除了RKE配置文件选项之外，还有可以在配置文件（YAML）中配置的Rancher特定设置):

#### docker_root_dir

参阅[Docker根目录](#docker-root-directory).

#### enable_cluster_monitoring

启用或禁用[集群监控](/docs/cluster-admin/tools/monitoring/)选项.

#### enable_network_policy

启用或禁用项目网络隔离选项。

#### local_cluster_auth_endpoint

参考[授权集群访问地址](#authorized-cluster-endpoint).

例子:

```yaml
local_cluster_auth_endpoint:
  enabled: true
  fqdn: 'FQDN'
  ca_certs: 'BASE64_CACERT'
```

#### 自定义网络插件

_v2.2.4可用_

您可以使用RKE的[用户定义附加功能]({{<baseurl>}}/rke/latest/en/config-options/add-ons/user-defined-add-ons/)添加自定义网络插件。 您可以在部署Kubernetes集群后定义要部署的任何附加组件。

有两种方法可以指定附加组件:

- [行内附加组件]({{<baseurl>}}/rke/latest/en/config-options/add-ons/user-defined-add-ons/#in-line-add-ons)
- [引用加载项的YAML文件]({{<baseurl>}}/rke/latest/en/config-options/add-ons/user-defined-add-ons/#referencing-yaml-files-for-add-ons)

有关如何通过编辑配置自定义网络插件的示例`cluster.yml`，请参阅[RKE文档。]({{<baseurl>}}/rke/latest/en/config-options/add-ons/network-plugins/custom-network-plugins-example)
