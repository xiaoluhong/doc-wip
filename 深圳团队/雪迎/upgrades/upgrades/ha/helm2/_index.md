---
title: 使用Helm 2升级Kubernetes上安装的Rancher
---

> 在发布Helm 3之后，[在Kubernetes集群上升级Rancher的说明](./ha)已更新为使用Helm 3。
>
> 如果您使用的是Helm 2，我们建议[迁移到Helm 3](https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/) ，因为它使用起来更简单，而且比Helm 2更安全.
>
> 本节提供了使用Helm 2升级Rancher的较旧说明的副本，适用于无法升级到Helm 3的情况。

以下说明将指导您使用Helm升级Kubernetes集群上安装的Rancher服务器。

要升级Kubernetes集群中的组件，或升级[Kubernetes服务] [Kubernetes services]({{<baseurl>}}/rke/latest/en/config-options/services/) 或 [add-ons]({{< baseurl >}}/rke/latest/en/config-options/add-ons/)，请参阅[RKE的升级文档]({{<baseurl>}}/rke/latest/en/upgrades/)， Rancher Kubernetes引擎。

如果您使用RKE附加组件yaml安装了Rancher，请按照[迁移或升级](/docs/upgrades/upgrades/migrating-from-rke-add-on)的说明进行操作。

> **注意:**
>
> - [Let's Encrypt 将从2019年11月1日开始阻止早于0.8.0的证书管理器实例。](https://community.letsencrypt.org/t/blocking-old-cert-manager-versions/98753) 按照[这些说明](/docs/installation/options/upgrading-cert-manager)升级证书到最新版本。
> - 如果要将Rancher从v2.x升级到v2.3 +，并且正在使用外部TLS终止，则需要将cluster.yml编辑为[使用转发的host headers.](/docs/installation/ha/helm-rancher/chart-options/#configuring-ingress-for-external-tls-when-using-nginx-v0-25)
> - 升级说明假定您使用的是Helm3。有关从Helm 2开始的安装迁移的信息，请参阅官方的[Helm 2到3迁移文档。](https://helm.sh/blog/migrate-from-helm-v2-to-helm-v3/)  [此节](/docs/upgrades/helm2) 提供了使用Helm 2的较旧升级说明的副本，如果无法升级到Helm 3，则可以使用该说明。

## 先决条件

- **从Rancher文档中 [已知的升级问题](/docs/upgrades/upgrades/#known-upgrade-issues) 和 [警告](/docs/upgrades/upgrades/#caveats)** 参照升级Rancher中最值得注意的问题。关于每个Rancher版本的已知问题的更完整的清单可以在[GitHub](https://github.com/rancher/rancher/releases) 和[Rancher forums](https://forums.rancher.com/c/announcements/12)找到。

- **对于 [仅用于离线安装](/docs/installation/other-installation-methods/air-gap) 收集并推送新的Rancher server版本镜像。** 请按照指南将你想要升级到Rancher版本的镜像 [推送到私有仓库](/docs/installation/other-installation-methods/air-gap/populate-private-registry/)。
## 升级大纲

请按照以下步骤升级Rancher服务器：

- [A. 备份正在运行Rancher服务器的Kubernetes集群](#a-backup-your-kubernetes-cluster-that-is-running-rancher-server)
- [B. 更新Helm chart仓库](#b-update-the-helm-chart-repository)
- [C. 升级 Rancher](#c-upgrade-rancher)
- [D. 验证升级](#d-verify-the-upgrade)

#### A. 备份运行Rancher Server的Kubernetes集群
为运行Rancher服务器的Kubernetes集群[拍摄一次快照](/docs/backups/backups/ha-backups/#option-b-one-time-snapshots)。
如果升级过程中出现问题，则将快照用作还原点。
#### B. 更新 Helm chart仓库

1. 更新本地的 helm 仓库

   ```bash
   helm repo update
   ```

1. 获取用于安装Rancher的仓库名称。

   有关仓库及其差异的信息，请参见[Helm Chart仓库](/docs/installation/options/server-tags/#helm-chart-repositories).

   {{< release-channel >}}

   ```bash
   helm repo list

   NAME          	       URL
   stable        	       https://kubernetes-charts.storage.googleapis.com
   rancher-<CHART_REPO>	 https://releases.rancher.com/server-charts/<CHART_REPO>
   ```

   > **注意:** 如果要切换到其他Helm chart仓库，请按照[如何切换仓库的步骤](/docs/installation/options/server-tags/#switching-to-a-different-helm-chart-repository)。如果切换仓库，请确保在继续执行第3步之前再次列出仓库，以确保添加了正确的仓库。

1) 从Helm chart仓库中获取最新的chart以安装Rancher。

   该命令将下拉最新的chart并将其保存为当前目录中的.tgz文件。


   ```plain
   helm fetch rancher-<CHART_REPO>/rancher
   ```

#### C. 升级 Rancher

本节介绍如何使用Helm升级Rancher的常规（连接Internet） 或离线安装。

 tabs
 tab "Kubernetes Upgrade"

从已安装的当前Rancher Helm chart中获取通过`--set`传递的值。

```bash
helm get values rancher

hostname: rancher.my.org
```

> **注意:** 此命令将列出更多的值。这只是其中一个值的示例。

如果您也将cert-manager从0.11.0之前的版本升级到最新版本，请遵循 `选项B：重新安装Rancher`。否则，请遵循`选项A：升级Rancher`。

 accordion label="Option A: Upgrading Rancher"

使用所有设置将Rancher升级到最新版本。

取上一步中的所有值，然后使用`--set key=value`将它们附加到命令中。> 注意: 上一步中将有更多选项需要附加。

```bash
helm upgrade rancher-<CHART_REPO>/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org
```

 /accordion

 accordion label="Option B: Reinstalling Rancher chart"

如果您当前正在运行版本低于v0.11的cert-manger，并且想将Rancher和cert-manager都升级到新版本，则由于API中的更改，您需要重新安装Rancher和cert-manger。 cert-manger v0.11。

请参阅[升级证书管理器](/docs/installation/options/upgrading-cert-manager) 页面以获取更多信息。
1. 卸载Rancher

   ```bash
   helm delete rancher -n cattle-system
   ```

2. 使用所有设置将Rancher重新安装到最新版本。获取上一步中的所有值，然后使用`--set key=value`. 将它们附加到命令中。注意：上一步中将有更多选项需要附加。

   ```bash
   helm install rancher-<CHART_REPO>/rancher \
   --name rancher \
   --namespace cattle-system \
   --set hostname=rancher.my.org
   ```

 /accordion

 /tab

 tab "Kubernetes Air Gap Upgrade"

1. 使用安装Rancher时使用的相同选择选项渲染Rancher模板。使用下面的参考表替换每个占位符。为了配置任何由Rancher启动的Kubernetes集群或Rancher工具，需要将Rancher配置为使用私有仓库。

   根据您在安装过程中所做的选择，完成以下过程之一。
   | 占位符                      | 描述                                     |
   | -------------------------------- | ----------------------------------------------- |
   | `<VERSION>`                      | 输出压缩包的版本号。      |
   | `<RANCHER.YOURDOMAIN.COM>`       | 您指向负载均衡器的DNS名称。 |
   | `<REGISTRY.YOURDOMAIN.COM:PORT>` | 您的私有仓库的DNS名称。        |
   | `<CERTMANAGER_VERSION>`          | 在k8s集群上运行的证书管理器版本。   |

 accordion id="self-signed" label="Option A-Default Self-Signed Certificate" 

```plain
helm template ./rancher-<VERSION>.tgz --output-dir . \
--name rancher \
--namespace cattle-system \
--set hostname=<RANCHER.YOURDOMAIN.COM> \
--set certmanager.version=<CERTMANAGER_VERSION> \
--set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher \
--set systemDefaultRegistry=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Available as of v2.2.0, set a default private registry to be used in Rancher
--set useBundledSystemChart=true # Available as of v2.3.0, use the packaged Rancher system charts
```

 /accordion
 accordion id="secret" label="Option B: Certificates From Files using Kubernetes Secrets" 

> **注意:** 如果您使用的是由私有CA签名的证书，请在 `--set ingress.tls.source=secret` 之后添加 `--set privateCA=true`。

```plain
helm template ./rancher-<VERSION>.tgz --output-dir . \
--name rancher \
--namespace cattle-system \
--set hostname=<RANCHER.YOURDOMAIN.COM> \
--set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher \
--set ingress.tls.source=secret \
--set systemDefaultRegistry=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Available as of v2.2.0, set a default private registry to be used in Rancher
--set useBundledSystemChart=true # Available as of v2.3.0, use the packaged Rancher system charts
```

 /accordion

2. 将渲染的清单目录复制到可以访问Rancher服务器集群的系统，然后应用渲染的模板。

   使用 `kubectl` 应用渲染的清单.

   ```plain
   kubectl -n cattle-system apply -R -f ./rancher
   ```

 /tab 
 /tabs 

#### D. 验证升级

登录到Rancher以确认升级成功。

> **升级后出现网络问题？**
>
> 请参阅[还原集群网络](/docs/upgrades/upgrades/namespace-migration/#restoring-cluster-networking).

### 回滚

如果出现问题，请按照[回滚](/docs/upgrades/rollbacks/ha-server-rollbacks/)的指示来还原执行升级之前拍摄的快照。
