---
title: 升级Docker安装的Rancher
---

以下说明将指导您升级Docker安装的Rancher Server。

## 先决条件

- 从Rancher文档中的 **[已知升级问题](/docs/upgrades/upgrades/#known-upgrade-issues) 和 [警告](/docs/upgrades/upgrades/#caveats)** 查看升级Rancher中最值得注意的问题. 可以在[GitHub](https://github.com/rancher/rancher/releases) 和 [Rancher forums.](https://forums.rancher.com/c/announcements/12)的发行说明中找到每个Rancher版本的已知问题的更完整列表。
- **对于 [air gap安装,](/docs/installation/other-installation-methods/air-gap) 收集并填充新Rancher server版本的镜像。** 请遵循指南以要升级到的Rancher版本的图像 [填充私有仓库](/docs/installation/other-installation-methods/air-gap/populate-private-registry/).

## 占位符

在升级过程中，您将输入一系列命令，用环境中的数据填充占位符。这些占位符用尖括号和所有大写字母(`<EXAMPLE>`)表示。

这是带有占位符的命令的**示例**：

```
docker stop <RANCHER_CONTAINER_NAME>
```

在此命令中，{`<RANCHER_CONTAINER_NAME>`} 是您的Rancher容器的名称。

请交叉参考下面的图像和参考表，以了解如何获取此占位符数据。在开始升级之前，记下或复制此信息。

<sup>终端 `docker ps` 命令，显示在何处查找 {`<RANCHER_CONTAINER_TAG>`} 和 {`<RANCHER_CONTAINER_NAME>`}</sup>
![占位符参考](/img/rancher/placeholder-ref.png)

| 占位符                  | 示例           | 描述                                               |
| ---------------------------- | ----------------- | --------------------------------------------------------- |
| {`<RANCHER_CONTAINER_TAG>`}  | `v2.1.3`          | 初始安装拉取的Rancher镜像。 |
| {`<RANCHER_CONTAINER_NAME>`} | `festive_mestorf` | Rancher容器的名称。                      |
| `<RANCHER_VERSION>`          | `v2.1.3`          | 创建备份的Rancher的版本。 |
| {`<DATE>`}                   | `2018-12-19`      | 数据容器或备份的创建日期。 |

<br/>

您可以通过远程连接登录Rancher server并输入命令以查看正在运行的容器：`docker ps`，从而获得 {`<RANCHER_CONTAINER_TAG>`} 和 {`<RANCHER_CONTAINER_NAME>`}。您还可以查看使用其他命令停止的容器：`docker ps -a`。创建备份期间，随时可以使用这些命令获得帮助。

## 升级大纲


在升级期间，您可以从当前Rancher容器中创建数据的副本，并在出现问题时进行备份。然后，您可以使用现有数据在新容器中部署新版本的Rancher。请按照以下步骤升级Rancher server：

- [A. 从Rancher server容器中创建数据的副本](#a-create-a-copy-of-the-data-from-your-rancher-server-container)
- [B. 创建备用压缩包](#b-create-a-backup-tarball)
- [C. 拉取新的Docker镜像](#c-pull-the-new-docker-image)
- [D. 启动新的Rancher server容器](#d-start-the-new-rancher-server-container)
- [E. 验证升级](#e-verify-the-upgrade)
- [F. 清理旧的Rancher server容器](#f-clean-up-your-old-rancher-server-container)

#### A. 从Rancher server容器中创建数据的副本

1. 使用远程终端连接，登录运行Rancher server的节点。

1. 停止当前正在运行的Rancher server的容器。将{`<RANCHER_CONTAINER_NAME>`} 替换为Rancher容器的名称。

   ```
   docker stop <RANCHER_CONTAINER_NAME>
   ```

1. <a id="backup"></a>使用下面的命令替换每个占位符，从刚刚停止的Rancher容器中创建一个数据容器。

   ```
   docker create --volumes-from <RANCHER_CONTAINER_NAME> --name rancher-data rancher/rancher:<RANCHER_CONTAINER_TAG>
   ```

#### B. 创建一个备用压缩包

1. <a id="tarball"></a>从您刚创建的数据容器(`rancher-data`), 创建一个备份压缩包 ({`rancher-data-backup-<RANCHER_VERSION>-<DATE>.tar.gz`}).

   如果升级期间出现问题，则此tarball将用作回滚点。使用以下命令，替换每个[占位符](#before-you-start).


    ```
    docker run --volumes-from rancher-data -v $PWD:/backup busybox tar zcvf /backup/rancher-data-backup-<RANCHER_VERSION>-<DATE>.tar.gz /var/lib/rancher
    ```

    **步骤结果:** 当您输入此命令时，应运行一系列命令。

1. 输入`ls`命令以确认已创建备份压缩包。它的名称类似于 {`rancher-data-backup-<RANCHER_VERSION>-<DATE>.tar.gz`}.

   ```
   [rancher@ip-10-0-0-50 ~]$ ls
   rancher-data-backup-v2.1.3-20181219.tar.gz
   ```

1. 将备份压缩包移到Rancher server外部的安全位置。


#### C. 拉取新的Docker映像

拉取要升级到的Rancher版本的映像。

| 占位符             | 描述                                                                                                    |
| ----------------------- | -------------------------------------------------------------------------------------------------------------- |
| `<RANCHER_VERSION_TAG>` | 您要升级到的[Rancher 版本](/docs/installation/options/server-tags/) 的发行标签。 |

```
docker pull rancher/rancher:<RANCHER_VERSION_TAG>
```

#### D. 启动新的Rancher Server容器

使用来自`rancher-data`容器的数据启动一个新的Rancher server容器。记住要传入启动原始容器时使用的所有环境变量。

> **重要提示:** _不要_ 在启动升级后停止升级，即使升级过程似乎比预期的要长。停止升级可能会导致将来升级期间数据库迁移错误。

如果使用代理，请参阅 [HTTP代理配置。](/docs/installation/other-installation-methods/single-node-docker/proxy/)

如果您配置了自定义CA根证书来访问服务，请参阅[自定义CA根证书。](/docs/installation/other-installation-methods/single-node-docker/advanced/#custom-ca-certificate)

如果您正在使用Rancher API记录所有交易，请参阅 [API 审计](/docs/installation/other-installation-methods/single-node-docker/advanced/#api-audit-log)

要查看启动新的Rancher server容器时要使用的命令，请从以下选项中选择：

- Docker 升级
- air gap安装的Docker升级

 tabs 
 tab "Docker Upgrade" 

选择您已安装Rancher server的选项

 accordion id="option-a" label="Option A-Default Self-Signed Certificate" 

如果选择使用Rancher生成的自签名证书，则在启动原始Rancher server容器的命令中添加`--volumes-from rancher-data`。

| 占位符             | 描述                                                                                                    |
| ----------------------- | -------------------------------------------------------------------------------------------------------------- |
| `<RANCHER_VERSION_TAG>` | 您要升级到的 [Rancher 版本](/docs/installation/options/server-tags/) 的发行标签。 |

```
docker run -d --volumes-from rancher-data \
  --restart=unless-stopped \
  -p 80:80 -p 443:443 \
	rancher/rancher:<RANCHER_VERSION_TAG>
```

 /accordion 

 accordion id="option-b" label="Option B-Bring Your Own Certificate: Self-Signed" 

如果您选择携带自己的自签名证书，则在启动原始Rancher server容器的命令中添加`--volumes-from rancher-data`，并需要访问与该证书相同的证书最初安装。

> **证书前提提示:** 证书文件必须为 [PEM 格式](/docs/installation/other-installation-methods/single-node-docker/#pem)。在您的证书文件中，包括链中的所有中间证书。请先使用证书订购证书，然后再订购中间体。有关示例，请参见 [SSL常见问题解答/故障排除](#cert-order).

| 占位符             | 描述                                                                                                    |
| ----------------------- | -------------------------------------------------------------------------------------------------------------- |
| `<CERT_DIRECTORY>`      | 包含证书文件的目录的路径。                                                   |
| `<FULL_CHAIN.pem>`      | 完整证书链的路径。                                                                    |
| `<PRIVATE_KEY.pem>`     | 证书私钥的路径。                                                             |
| `<CA_CERTS>`            | 证书颁发机构的证书的路径。                                                          |
| `<RANCHER_VERSION_TAG>` | 您要升级到的 [Rancher 版本](/docs/installation/options/server-tags/) 的发行标签。
 |

```
docker run -d --volumes-from rancher-data \
  --restart=unless-stopped \
	-p 80:80 -p 443:443 \
	-v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
	-v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
	-v /<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \
	rancher/rancher:<RANCHER_VERSION_TAG>
```

 /accordion 
 accordion id="option-c" label="Option C-Bring Your Own Certificate: Signed by Recognized CA" 

如果选择使用由公认的CA签名的证书，则将 `--volumes-from rancher-data` 添加到启动原始Rancher server容器的命令中，并且需要访问与您相同的证书最初安装的。请记住，要在容器中包含`--no-cacerts`作为参数，以禁用Rancher生成的默认CA证书。

>  **证书前提条件提示:** 证书文件必须为 [PEM 格式](/docs/installation/other-installation-methods/single-node-docker/#pem)。在您的证书文件中， ，包括公认的CA提供的所有中间证书。请先使用证书订购证书，然后再订购中间体。有关示例，请参见[SSL常见问题解答/故障排除](#cert-order).

| 占位符             | 描述                                                                                                    |
| ----------------------- | -------------------------------------------------------------------------------------------------------------- |
| `<CERT_DIRECTORY>`      | 包含证书文件的目录的路径。                                                   |
| `<FULL_CHAIN.pem>`      | 完整证书链的路径。                                                                    |
| `<PRIVATE_KEY.pem>`     | 证书私钥的路径。                                                             |
| `<RANCHER_VERSION_TAG>` | 您要升级到的 [Rancher 版本](/docs/installation/options/server-tags/) 的发行标签。
 |

```
docker run -d --volumes-from rancher-data \
  --restart=unless-stopped \
	-p 80:80 -p 443:443 \
 	-v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
	-v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
	rancher/rancher:<RANCHER_VERSION_TAG> \
  --no-cacerts
```

 /accordion 
 accordion id="option-d" label="Option D-Let's Encrypt Certificate" 

> **记住:** Let's Encrypt为请求新证书提供速率限制。因此，请限制创建或销毁容器的频率。有关更多信息，请参阅 [关于速率限制的加密文档](https://letsencrypt.org/docs/rate-limits/).

如果您选择使用 [Let's Encrypt](https://letsencrypt.org/) 证书，则将`--volumes-from rancher-data`添加到启动原始Rancher server容器的命令中，并且需要提供最初安装Rancher时使用的域。

> **证书前提条件提示:**
>
> - 在您的DNS中创建一条记录，将您的Linux主机IP地址绑定到您要用于Rancher访问的主机名（例如，`rancher.mydomain.com`）。
> - 在Linux主机上打开端口 `TCP/80`。The Let's Encrypt http-01 challenge 可以来自任何源IP地址，因此端口`TCP/80`必须对所有IP地址开放。

| 占位符             | 描述                                                                                                    |
| ----------------------- | -------------------------------------------------------------------------------------------------------------- |
| `<RANCHER_VERSION_TAG>` | 您要升级到的 [Rancher 版本](/docs/installation/options/server-tags/) 的发行标签。|
| `<YOUR.DNS.NAME>`       | 您最初开始使用的域地址                                                      |

```
docker run -d --volumes-from rancher-data \
  --restart=unless-stopped \
	-p 80:80 -p 443:443 \
	rancher/rancher:<RANCHER_VERSION_TAG> \
  --acme-domain <YOUR.DNS.NAME>
```

 /accordion 

 /tab 
 tab "Docker Air Gap Upgrade" 

为了安全起见，使用Rancher时需要SSL（安全套接字层）。 SSL保护所有Rancher网络通信的安全，例如在您登录集群或与集群交互时。

> 对于从v2.2.0到v2.2.x的Rancher版本，您需要将`system-charts`存储库镜像到网络中Rancher可以访问的位置。然后，在安装Rancher之后，您将需要配置Rancher以使用该存储库。有关详细信息，请参考 [在v2.3.0之前为Rancher设置系统图表。](/docs/installation/options/local-system-charts/#setting-up-system-charts-for-rancher-prior-to-v2-3-0)

启动新的Rancher server容器时，请从以下选项中选择：
 accordion id="option-a" label="Option A-Default Self-Signed Certificate" 

如果选择使用Rancher生成的自签名证书，则在启动原始Rancher server容器的命令中添加`--volumes-from rancher-data`。

| 占位符                      | 描述                                                                                                       |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` | 您的私有仓库URL和端口。                                                                          |
| `<RANCHER_VERSION_TAG>`          | 您要升级到的[Rancher 版本](/docs/installation/options/server-tags/) 的发行标签。 |

```
  docker run -d --volumes-from rancher-data \
      --restart=unless-stopped \
      -p 80:80 -p 443:443 \
      -e CATTLE_SYSTEM_DEFAULT_REGISTRY=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Set a default private registry to be used in Rancher
      -e CATTLE_SYSTEM_CATALOG=bundled \ #Available as of v2.3.0, use the packaged Rancher system charts
      <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
```

 /accordion 

 accordion id="option-b" label="Option B-Bring Your Own Certificate: Self-Signed" 

如果您选择携带自己的自签名证书，则在启动原始Rancher server容器的命令中添加`--volumes-from rancher-data`，并需要访问与该证书相同的证书最初安装。

> **前提条件的提醒:** 证书文件必须为[PEM 格式](/docs/installation/other-installation-methods/single-node-docker/#pem)。在您的证书文件中，包括链中的所有中间证书。请先使用证书订购证书，然后再订购中间体。有关示例，请参见 [SSL常见问题解答/故障排除](#cert-order).

| 占位符                      | 描述                                                                                                    |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `<CERT_DIRECTORY>`               | 包含证书文件的目录的路径。                                                   |
| `<FULL_CHAIN.pem>`               | 完整证书链的路径。                                                              |
| `<PRIVATE_KEY.pem>`              | 证书私钥的路径。                                                             |
| `<CA_CERTS>`                     | 证书颁发机构的证书的路径。                                                         |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` |您的私有仓库URL和端口。                                                                                       |
| `<RANCHER_VERSION_TAG>`          | 您要升级到的[Rancher 版本](/docs/installation/options/server-tags/) 的发行标签。 |

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


如果选择使用由公认的CA签名的证书，则将`--volumes-from rancher-data`添加到启动原始Rancher server容器的命令中，并且需要访问与您相同的证书最初安装的。
> **前提条件的提醒:** 证书文件必须为[PEM 格式](/docs/installation/other-installation-methods/single-node-docker/#pem)。在您的证书文件中，包括公认的CA提供的所有中间证书。请先使用证书订购证书，然后再订购中间体。有关示例，请参见[SSL常见问题解答/故障排除](#cert-order)。

| 占位符                      | 描述                                                                                                    |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `<CERT_DIRECTORY>`               | 包含证书文件的目录的路径。                                                   |
| `<FULL_CHAIN.pem>`               | 完整证书链的路径。                                                              |
| `<PRIVATE_KEY.pem>`              | 证书私钥的路径。                                                             |
| `<REGISTRY.YOURDOMAIN.COM:PORT>` |您的私有仓库URL和端口。                                                                                       |
| `<RANCHER_VERSION_TAG>`          | 您要升级到的[Rancher 版本](/docs/installation/options/server-tags/) 的发行标签。 |

> **注意:** 使用`--no-cacerts`作为容器的参数来禁用Rancher生成的默认CA证书。

```
docker run -d --volumes-from rancher-data \
    --restart=unless-stopped \
     -p 80:80 -p 443:443 \
     --no-cacerts \
     -v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
     -v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
     -e CATTLE_SYSTEM_DEFAULT_REGISTRY=<REGISTRY.YOURDOMAIN.COM:PORT> \ # Set a default private registry to be used in Rancher
     -e CATTLE_SYSTEM_CATALOG=bundled \ #Available as of v2.3.0, use the packaged Rancher system charts
     <REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
```

 /accordion 
 /tab 
 /tabs 

**结果:** 您已升级Rancher。现在，已升级服务器中的数据将保存到`rancher-data`容器中，以用于将来的升级。

#### E. 验证升级

登录到Rancher。通过检查浏览器窗口左下角显示的版本，确认升级成功.

> **升级后您的用户集群中有网络问题吗？**
>
> 请参阅 [还原集群网络](/docs/upgrades/upgrades/namespace-migration/#restoring-cluster-networking).

#### F. 清理旧的Rancher Server容器

删除以前的Rancher server容器。如果仅停止上一个Rancher server容器（并且不删除它），则该容器可能会在下一个服务器重启后重新启动。

### 回滚

如果升级未成功完成，则可以将Rancher server及其数据回滚到最后的正常状态。有关更多信息，请参阅 [Docker 回滚](/docs/upgrades/rollbacks/single-node-rollbacks/).
