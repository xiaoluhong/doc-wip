---
title: '2. 收集镜像并将其推送到您的私有镜像仓库'
---

> **先决条件:** 您必须具有可用的[私有镜像仓库](https://docs.docker.com/registry/deploying/)。
>
> **注意:** 镜像推送到私有镜像仓库与安装 HA 和 Docker 的过程相同，本节中的区别基于您是否计划支持 Windows 群集。

默认情况下，用于[配置 Kubernetes clusters](/docs/cluster-provisioning/) 或在 Rancher 中使用到的[工具](/docs/tools/)所需要的镜像，例如 monitoring, pipelines, alerts, 都是从 Docker Hub 中提取的. 在离线环境下安装 Rancher, 您将需要一个私有镜像仓库, 确保 Rancher server 可以访问该私有镜像仓库。然后，您将从私有镜像仓库中拉取所有镜像。

本节介绍如何设置私有镜像仓库,以便在安装 Rancher 时，Rancher 可以从此镜像仓库中拉取所有必需的镜像。

默认情况下，假设您仅支持 Linux 群集, 我们将提供如何推送镜像到私有镜像仓库的步骤, 但是如果您打算支持 [Windows clusters](/docs/cluster-provisioning/rke-clusters/windows-clusters/),则有单独的说明来支持 Windows 群集。

 tabs 
 tab "Linux Only Clusters" 

对于仅需支持 Linux 群集的 Rancher server，这些是填充私有镜像仓库的步骤。

A. 查找您需要的 Rancher 版本 <br />
B. 收集所有必需的镜像 <br />
C. 将镜像保存在您的主机上 <br />
D. 推送镜像到您的私有镜像仓库

#### 先决条件

这些步骤要求您可以访问互联网，可访问到您的私有镜像仓库以及至少 20GB 磁盘空间的Linux主机

#### A. 查找您需要的 Rancher 版本

1. 访问 [releases page](https://github.com/rancher/rancher/releases) 找到您要安装的 Rancher 版本 v2.x.x release。 不要下载标记为 `rc` 或者 `Pre-release` 的版本，因为它们对于生产环境而言不稳定。

2. 从 release's **Assets** 部分 (如上图所示), 下载以下文件，这些文件是在离线环境下安装 Rancher 所必需的:

| Release File             | Description                                                                                                                          |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| `rancher-images.txt`     | This file contains a list of images needed to install Rancher, provision clusters and user Rancher tools.                            |
| `rancher-save-images.sh` | This script pulls all the images in the `rancher-images.txt` from Docker Hub and saves all of the images as `rancher-images.tar.gz`. |
| `rancher-load-images.sh` | This script loads images from the `rancher-images.tar.gz` file and pushes them to your private registry.                             |

#### B. 收集所有必需的镜像 (对于使用Rancher生成的自签名证书进行的Kubernetes安装)

在 Kubernetes 安装中，如果您选择使用 Rancher 默认的自签名TLS证书， 则 `rancher-images.txt` 还必须添加 [`cert-manager`](https://hub.helm.sh/charts/jetstack/cert-manager) 镜像。如果您使用自己的证书，则可以跳过此步骤。

1.  下载最新的 `cert-manager` 并解析模板以获取镜像详细信息：

    > **注意:** 最近对证书管理器的更改需要升级。如果要升级 Rancher 并使用 v0.9.1 之前的 cert-manager 版本，请参阅 [upgrade documentation](/docs/installation/options/upgrading-cert-manager/).

    ```plain
    helm repo add jetstack https://charts.jetstack.io
    helm repo update
    helm fetch jetstack/cert-manager --version v0.9.1
    helm template ./cert-manager-<version>.tgz | grep -oP '(?<=image: ").*(?=")' >> ./rancher-images.txt
    ```

2.  对镜像列表进行排序和去重, 用来消除数据源的重复:

    ```plain
    sort -u rancher-images.txt -o rancher-images.txt
    ```

#### C. 将镜像保存在您的主机上

1. 制作 `rancher-save-images.sh` 可执行文件:

   ```
   chmod +x rancher-save-images.sh
   ```

1. 运行 `rancher-save-images.sh` 与 `rancher-images.txt` 镜像列表创建所需的所有镜像的压缩包:
   ```plain
   ./rancher-save-images.sh --image-list ./rancher-images.txt
   ```
   **结果:** Docker开始拉取用于离线安装 Rancher 的所有镜像。 耐心点。此过程需要几分钟。该过程完成后，当前目录将输出名为的tarball rancher-images.tar.gz。检查输出是否在目录中。

#### D. 推送镜像到您的私有镜像仓库

使用脚本将 `rancher-images.tar.gz` 中的镜像推送到您的私有镜像仓库，保证 `rancher-images.txt` 和 `rancher-load-images.sh` 在同一工作目录。

1.  登录到您的私有镜像仓库:
    ```plain
    docker login <REGISTRY.YOURDOMAIN.COM:PORT>
    ```
1.  制作 `rancher-load-images.sh` 可执行文件：

    ```
    chmod +x rancher-load-images.sh
    ```

1.  使用 `rancher-load-images.sh` 自动执行 extract， tag, push `rancher-images.txt` 和 `rancher-images.tar.gz` 到您的私有镜像仓库：
    `plain ./rancher-load-images.sh --image-list ./rancher-images.txt --registry <REGISTRY.YOURDOMAIN.COM:PORT>`
     /tab 
     tab "Linux and Windows Clusters" 

_Available as of v2.3.0_

对与同时支持 Linux 和 Windows 集群的 Rancher servers，会有不同的步骤来为私有镜像仓库填充 Windows 镜像和 Linux 镜像的. 由于 Windows 群集是Linux 和 Windows混合的节点，所以 Linux 镜像只是一个推送到私有Registry的清单。

#### Windows Steps

需要从 Windows server 工作站收集并推送 Windows 镜像。

A. 查找您需要的 Rancher 版本 <br />
B. 将镜像保存到 Windows Server 工作站 <br />
C. 准备 Docker 守护程序 <br />
D. 填充私有镜像仓库

 accordion label="将 Windows 镜像填充到私有镜像仓库"

#### 先决条件

执行这些步骤的前提需要您使用 Windows Server 1809 工作站， 有权访问到您的私有镜像仓库以及至少50 GB的磁盘空间。

工作站必须安装 Docker 18.02+ 才能支持清单，在配置 Windows 群集时需要这些清单。

#### A. 查找您需要的 Rancher 版本

1. 访问 [releases page](https://github.com/rancher/rancher/releases) 找到您要安装的 Rancher 版本 v2.x.x release。不要下载标记为 `rc` 或者 `Pre-release` 的版本，因为它们对于生产环境而言不稳定。

2. 从 release's **Assets** 部分 (如上图所示)，下载以下文件：

   | Release File                 | Description                                                                                                                                          |
   | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
   | `rancher-windows-images.txt` | This file contains a list of Windows images needed to provision Windows clusters.                                                                    |
   | `rancher-save-images.ps1`    | This script pulls all the images in the `rancher-windows-images.txt` from Docker Hub and saves all of the images as `rancher-windows-images.tar.gz`. |
   | `rancher-load-images.ps1`    | This script loads the images from the `rancher-windows-images.tar.gz` file and pushes them to your private registry.                                 |

#### B. 将镜像保存到 Windows Server 工作站

1. 使用 `powershell`，跳转到包含在上一步中下载的文件的目录。

1. 运行 `rancher-save-images.ps1` 以创建所有必需镜像的压缩包：

   ```plain
   ./rancher-save-images.ps1
   ```

   **结果:** Docker开始拉取用于离线安装 Rancher 的所有镜像。耐心点。此过程需要几分钟。该过程完成后，当前目录将输出名为的tarball rancher-windows-images.tar.gz。检查输出是否在目录中。

#### C. Prepare the Docker daemon

1. 将您的私有镜像仓库地址附加到Docker 守护程序 (`C:\ProgramData\Docker\config\daemon.json`) 中的 `allow-nondistributable-artifacts` config字段。由于Windows镜像的基本镜像由 mcr.microsoft.com registry 维护，因此此步骤是必需的，因为 Docker Hub 中缺少 Microsoft registry 中的各层，因此需要将其拉到私有镜像仓库中。

   ```json
   {
     ...
     "allow-nondistributable-artifacts": [
       ...
       "<REGISTRY.YOURDOMAIN.COM:PORT>"
     ]
     ...
   }
   ```

#### D. 填充私人镜像仓库

使用脚本将 `rancher-windows-images.tar.gz` 中的镜像移动到您的私有镜像仓库，保证 `rancher-windows-images.txt` 和 `rancher-load-images.sh` 在同一工作目录。

1. 使用 `powershell`，如果需要，登录到您的私有镜像仓库：

   ```plain
   docker login <REGISTRY.YOURDOMAIN.COM:PORT>
   ```

1. 使用 `powershell`，`rancher-load-images.ps1` 自动执行 extract，tag，push 从 `rancher-images.tar.gz` 中提取镜像到您的私有镜像仓库：

   ```plain
   ./rancher-load-images.ps1 --registry <REGISTRY.YOURDOMAIN.COM:PORT>
   ```

 /accordion 

#### Linux Steps

需要从 Linux 主机收集并推送 Linux 镜像，必须在将 Windows 镜像填充到私有镜像仓库之后进行。需要执行的步骤不同于仅在 Linux 环境下部署 Rancher,因为需要推送的 Linux 镜像既支持 Linux 又支持 Windows

A. 查找您需要的 Rancher 版本 <br />
B. 收集所有必需的镜像 <br />
C. 将镜像保存在您的主机上 <br />
D. 推送镜像到您的私有镜像仓库

 accordion label="将 Linux 镜像收集并填充到私有镜像仓库" 

#### 先决条件

在使用 Linux 镜像填充私有镜像仓库之前，必须先用 Windows 镜像填充私有镜像仓库。如果您已经用 Linux 镜像填充了私有镜像仓库，您需要遵循一下说明，因为它们将发布支持 Windows 和 Linux 镜像的清单

要求您的 Linux 工作站具有Internet访问权限，可以访问您的私有镜像仓库以及至少20 GB磁盘空间。

工作站必须安装 Docker 18.02+ 才能支持清单，在配置 Windows 群集时需要这些清单。

#### A. 查找您需要的 Rancher 版本

1. 访问 [releases page](https://github.com/rancher/rancher/releases) 找到您要安装的 Rancher 版本 v2.x.x release。不要下载标记为 `rc` 或者 `Pre-release` 的版本, 因为它们对于生产环境而言不稳定。

2. 从 release's **Assets** 部分 (如上图所示)，下载以下文件，这些文件是在离线环境下安装 Rancher 所必需的：

   | Release File                 | Description                                                                                                                          |
   | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
   | `rancher-images.txt`         | This file contains a list of images needed to install Rancher, provision clusters and user Rancher tools.                            |
   | `rancher-windows-images.txt` | This file contains a list of images needed to provision Windows clusters.                                                            |
   | `rancher-save-images.sh`     | This script pulls all the images in the `rancher-images.txt` from Docker Hub and saves all of the images as `rancher-images.tar.gz`. |
   | `rancher-load-images.sh`     | This script loads images from the `rancher-images.tar.gz` file and pushes them to your private registry.                             |

#### B. 收集所有必需的镜像

1. **对于使用Rancher生成的自签名证书进行的Kubernetes安装:** 在 Kubernetes 安装中，如果您选择使用 Rancher 默认的自签名TLS证书， 则 `rancher-images.txt` 还必须添加 [`cert-manager`](https://hub.helm.sh/charts/jetstack/cert-manager) 镜像。如果您使用自己的证书，则可以跳过此步骤。

   1. 下载最新的 `cert-manager` 并解析模板以获取镜像详细信息：

      > **Note:** Recent changes to cert-manager require an upgrade. If you are upgrading Rancher and using a version of cert-manager older than v0.9.1, please see our [upgrade documentation](/docs/installation/options/upgrading-cert-manager/)。

      ```plain
      helm repo add jetstack https://charts.jetstack.io
      helm repo update
      helm fetch jetstack/cert-manager --version v0.9.1
      helm template ./cert-manager-<version>.tgz | grep -oP '(?<=image: ").*(?=")' >> ./rancher-images.txt
      ```

   2. 对镜像列表进行排序和去重，用来消除数据源的重复：

      ```plain
      sort -u rancher-images.txt -o rancher-images.txt
      ```

#### C. 将镜像保存在您的主机上

1. 制作 `rancher-save-images.sh` 可执行文件：

   ```
   chmod +x rancher-save-images.sh
   ```

1. 运行 `rancher-save-images.sh` 与 `rancher-images.txt` 镜像列表创建所需的所有镜像的压缩包：

   ```plain
   ./rancher-save-images.sh --image-list ./rancher-images.txt
   ```

   **结果:** Docker开始拉取用于离线安装 Rancher 的所有镜像。耐心点。此过程需要几分钟。该过程完成后，当前目录将输出名为的tarball rancher-images.tar.gz。检查输出是否在目录中。

#### D. 推送镜像到您的私有镜像仓库

使用脚本将 `rancher-images.tar.gz` 中的镜像推送到您的私有镜像仓库，保证 `rancher-images.txt` 和 `rancher-load-images.sh` 在同一工作目录。

1. 登录到您的私有镜像仓库：

   ```plain
   docker login <REGISTRY.YOURDOMAIN.COM:PORT>
   ```

1. 制作 `rancher-load-images.sh` 可执行文件：

   ```
   chmod +x rancher-load-images.sh
   ```

1. 使用 `rancher-load-images.sh` 自动执行 extract, tag, push `rancher-images.txt` 到您的私有镜像仓库：

   ```plain
   ./rancher-load-images.sh --image-list ./rancher-images.txt \
     --windows-image-list ./rancher-windows-images.txt \
     --registry <REGISTRY.YOURDOMAIN.COM:PORT>
   ```

 /accordion 

 /tab 
 /tabs 

#### [Next: Kubernetes Installs - Launch a Kubernetes Cluster with RKE](/docs/installation/other-installation-methods/air-gap/launch-kubernetes/)

#### [Next: Docker Installs - Install Rancher](/docs/installation/other-installation-methods/air-gap/install-rancher/)
