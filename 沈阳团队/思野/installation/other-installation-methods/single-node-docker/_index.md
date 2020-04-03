---
title: 使用 Docker 在单节点上安装 Rancher
---

仅对于开发和测试环境，可以通过运行单个 Docker 容器来安装 Rancher。

在这种安装方案中，您将 Docker 安装在单个 Linux 主机上，然后使用单个 Docker 容器在您的主机上部署 Rancher。

> **是否使用外部 Load Balancer ?**
> 请参阅[使用外部负载均衡进行Docker安装](/docs/installation/other-installation-methods/single-node-docker/single-node-install-external-lb)。

## OS, Docker, Hardware, Networking 要求

确保您的节点满足常规[安装要求](/docs/installation/requirements/)。

## 1. 提供 Linux 主机

根据我们的[要求](/docs/installation/requirements)配置一个Linux主机，以启动Rancher服务器。

## 2. 选择一个 SSL 选项并安装 Rancher

为了安全起见，使用Rancher时需要 SSL（ecure Sockets Layer）。SSL 保护所有 Rancher 网络通信的安全，例如在您登录集群或与集群交互时。

> **Do you want to...**
>
> - 使用代理? 请参阅 [HTTP Proxy Configuration](/docs/installation/other-installation-methods/single-node-docker/proxy/)
> - 配置自定义 CA 根证书以访问您的服务? 请参阅 [Custom CA root certificate](/docs/installation/other-installation-methods/single-node-docker/advanced/#custom-ca-certificate/)
> - 完成离线环境下安装 Rancher? 请参阅 [Air Gap: Docker Install](/docs/installation/air-gap-single-node/)
> - 查看所有 Rancher API 的记录? 请参阅 [API Auditing](#api-audit-log)

Choose from the following options:

#### Option A: 由 Rancher 生成的默认自签名证书

如果要在不涉及身份验证的开发或测试环境中安装 Rancher，请使用其生成的自签名证书安装 Rancher。此安装选项省去了自己生成证书的麻烦。

登录到 Linux 主机，然后运行下面最简洁安装命令。

```bash
docker run -d --restart=unless-stopped \
	-p 80:80 -p 443:443 \
	rancher/rancher:latest
```

#### Option B: 使用主机的证书，自签名

在您的团队将要访问 Rancher 服务器的开发或测试环境中，创建一个自签名证书以供您的安装使用，以便您的团队可以验证他们正在连接到 Rancher 实例。

> **先决条件:**
> 使用 [OpenSSL](https://www.openssl.org/) 或您选择的其他方法创建自签名证书。
>
> - 证书文件必须为 [PEM 格式](#pem)。
> - 在您的证书文件中，包括链中的所有中间证书。请先使用证书订购证书，然后再订购中间体。有关示例，请参阅 [SSL FAQ / Troubleshooting](#cert-order)。

创建证书后，运行下面的 Docker 命令安装 Rancher。使用该 -v 标志并提供证书的路径，以将其挂载到容器中。

| Placeholder         | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `<CERT_DIRECTORY>`  | The path to the directory containing your certificate files. |
| `<FULL_CHAIN.pem>`  | The path to your full certificate chain.                     |
| `<PRIVATE_KEY.pem>` | The path to the private key for your certificate.            |
| `<CA_CERTS>`        | The path to the certificate authority's certificate.         |

```bash
docker run -d --restart=unless-stopped \
	-p 80:80 -p 443:443 \
	-v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
	-v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
	-v /<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \
	rancher/rancher:latest
```

#### Option C: 使用由证书签证机关（CA）签发的证书

在要公开展示应用程序的生产环境中，请使用由公认的CA签名的证书，这样您的用户群就不会遇到安全警告。

> **先决条件:**
>
> - 证书文件必须为[PEM 格式](#pem)。
> - 在您的证书文件中，包括链中的所有中间证书。请先使用证书订购证书，然后再订购中间体。有关示例，请参阅[SSL FAQ / Troubleshooting](#cert-order)。

获得证书后，运行下面的Docker命令。

- 使用该 -v 标志并提供证书的路径，以将其挂载到容器中，由于您的证书是由公认的 CA 签名的，因此不需要安装其他 CA 证书文件。
- 使用 `--no-cacerts` 容器的as参数禁用由 Rancher 生成的默认 CA 证书。
| Placeholder         | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `<CERT_DIRECTORY>`  | The path to the directory containing your certificate files. |
| `<FULL_CHAIN.pem>`  | The path to your full certificate chain.                     |
| `<PRIVATE_KEY.pem>` | The path to the private key for your certificate.            |

```bash
docker run -d --restart=unless-stopped \
	-p 80:80 -p 443:443 \
	-v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
	-v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
	rancher/rancher:latest \
	--no-cacerts
```

#### Option D: Let's Encrypt 证书

> **切记:** Let's Encrypt 证书为请求新证书提供了速率限制。因此，请限制创建或销毁容器的频率。有关更多信息，请参阅[Let's Encrypt documentation on rate limits](https://letsencrypt.org/docs/rate-limits/).

对于生产环境，您还可以选择使用 [Let's Encrypt](https://letsencrypt.org/) 证书. Let's Encrypt 使用 http-01 challenge来验证您对域的控制权. 您可以通过将要用于Rancher访问的主机名（例如rancher.mydomain.com）指向运行该计算机的 IP 来确认您控制该域。您可以通过在 DNS 中创建 A 记录来将主机名绑定到 IP 地址。


> **Prerequisites:**
>
> - Let's Encrypt证书是一项Internet服务. 因此，不能在离线环境下使用。
> - 在 DNS 中创建一条记录，该记录将 Linux 主机 IP 地址绑定到要用于 Rancher 访问的主机名 (`rancher.mydomain.com` 例如)。
> - `TCP/80` 在Linux主机上打开端口. Let's Encrypt 使用 http-01 challenge 可以来自任何源IP地址, 因此端口 `TCP/80` 必须开放给所有IP地址。

满足前提条件后，可以通过运行以下命令使用 Let's Encrypt 证书安装R ancher。

| Placeholder       | Description         |
| ----------------- | ------------------- |
| `<YOUR.DNS.NAME>` | Your domain address |

```
docker run -d --restart=unless-stopped \
	-p 80:80 -p 443:443 \
	rancher/rancher:latest \
	--acme-domain <YOUR.DNS.NAME>
```

### 高级选项

使用Docker在单个节点上安装Rancher时，可以启用几个高级选项：

- Custom CA Certificate
- API Audit Log
- TLS Settings
- Air Gap
- Persistent Data
- Running rancher/rancher and rancher/rancher-agent on the Same Node

有关详情，请参阅[此页面](./advanced)。

### 故障排除

请参阅[此页面](./troubleshooting)以获取常见问题和故障排除提示.

### What's Next?

- **推荐:** 查看[单节点备份和还原](/docs/installation/backups-and-restoration/single-node-backup-and-restoration/)。尽管您现在没有任何数据需要备份，但是我们建议您在常规Rancher 使用之后创建备份。
- 创建Kubernetes集群: [Provisioning Kubernetes Clusters](/docs/cluster-provisioning/)。
