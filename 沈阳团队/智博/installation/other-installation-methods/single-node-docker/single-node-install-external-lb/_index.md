---
title: Docker Install with External Load Balancer
---
针对Docker安装使用外部负载均衡器

For development and testing environments that have a special requirement to terminate TLS/SSL at a load balancer instead of your Rancher Server container, deploy Rancher and configure a load balancer to work with it conjunction. This install procedure walks you through deployment of Rancher using a single container, and then provides a sample configuration for a layer 7 Nginx load balancer.
对于需要在负载均衡器而不是Rancher Server容器处终止TLS / SSL的特殊开发和测试环境，请部署Rancher并配置负载均衡器以与其配合使用。 此安装过程将引导您使用单个容器完成Rancher的部署，然后为7层Nginx负载均衡器提供示例配置。

> **Want to skip the external load balancer?**
> See [Docker Installation](/docs/installation/single-node) instead.

> 不想使用外部负载均衡器
> 参考 [基于Docker安装](/docs/installation/single-node)

### Requirements for OS, Docker, Hardware, and Networking
关于操作系统、Docker、硬件、网络的需求

Make sure that your node fulfills the general [installation requirements.](/docs/installation/requirements/)
确保您的节点满足常规的[安装要求。](/docs/installation/requirements/)

### Installation Outline
安装概要

<!-- TOC -->

- [1. Provision Linux Host](#1-provision-linux-host)
- [2. Choose an SSL Option and Install Rancher](#2-choose-an-ssl-option-and-install-rancher)
- [3. Configure Load Balancer](#3-configure-load-balancer)

- [1. 设置Linux主机](#1-provision-linux-host)
- [2. SSL配置选项和Rancher安装](#2-choose-an-ssl-option-and-install-rancher)
- [3. 配置负载均衡器](#3-configure-load-balancer)
<!-- /TOC -->

### 1. Provision Linux Host
设置Linux主机

Provision a single Linux host according to our [Requirements](/docs/installation/requirements) to launch your {{< product >}} Server.
根据Rancher的[要求](/docs/installation/requirements)设置一台Linux主机，以启动{{<产品>}}服务器。

### 2. Choose an SSL Option and Install Rancher
SSL配置选项和Rancher安装

For security purposes, SSL (Secure Sockets Layer) is required when using Rancher. SSL secures all Rancher network communication, like when you login or interact with a cluster.
为了安全起见，使用Rancher时需要SSL。 SSL保护所有Rancher网络通信的安全，例如在您登录集群或与集群交互时。

> **Do you want to...**
>
> - Complete an Air Gap Installation?
> - 私有环境安装？
> - Record all transactions with the Rancher API?
> - 记录API审计信息？
>
> See [Advanced Options](#advanced-options) below before continuing.
> 参考[高级选项](#advanced-options)

Choose from the following options:
选择下面的选项：

 accordion id="option-a" label="Option A-Bring Your Own Certificate: Self-Signed" 
 选项A - 使用自签名证明
If you elect to use a self-signed certificate to encrypt communication, you must install the certificate on your load balancer (which you'll do later) and your Rancher container. Run the Docker command to deploy Rancher, pointing it toward your certificate.
如果选择使用自签名证书来加密通信，则必须在负载均衡器（稍后再做）和Rancher容器上安装证书。 运行Docker命令以部署Rancher，将其指向您的证书。

> **Prerequisites:**
> Create a self-signed certificate.
>
> - The certificate files must be in [PEM format](#pem).

> **先决条件：**
> 创建自签名证书
>
> - 证书必须是 [PEM 格式](#pem).

**To Install Rancher Using a Self-Signed Cert:**
基于自签名证书安装Rancher

1. While running the Docker command to deploy Rancher, point Docker toward your CA certificate file.
1.在运行Docker命令部署Rancher时，将Docker指向您的CA证书文件。

   ```
   docker run -d --restart=unless-stopped \
     -p 80:80 -p 443:443 \
     -v /etc/your_certificate_directory/cacerts.pem:/etc/rancher/ssl/cacerts.pem \
     rancher/rancher:latest
   ```

 /accordion 
 accordion id="option-b" label="Option B-Bring Your Own Certificate: Signed by Recognized CA" 
 选项B - 使用机构颁发的证书
If your cluster is public facing, it's best to use a certificate signed by a recognized CA.
如果您的集群是面向公众的，则最好使用由公认的CA签名的证书。

> **Prerequisites:**
>
> - The certificate files must be in [PEM format](#pem).

**To Install Rancher Using a Cert Signed by a Recognized CA:**
基于机构颁发证书安装Rancher

If you use a certificate signed by a recognized CA, installing your certificate in the Rancher container isn't necessary. We do have to make sure there is no default CA certificate generated and stored, you can do this by passing the `--no-cacerts` parameter to the container.
如果您使用由公认的CA签署的证书，则无需在Rancher容器中安装证书。 我们必须确保没有生成和存储默认的CA证书，您可以通过将`--no-cacerts`参数传递给容器来实现。

1.  Enter the following command.

        ```
        docker run -d --restart=unless-stopped \
        -p 80:80 -p 443:443 \
        rancher/rancher:latest --no-cacerts
        ```

     /accordion 

### 3. Configure Load Balancer

When using a load balancer in front of your Rancher container, there's no need for the container to redirect port communication from port 80 or port 443. By passing the header `X-Forwarded-Proto: https` header, this redirect is disabled.
在Rancher容器前面使用负载平衡器时，不需要该容器从端口80或端口443重定向端口通信。通过传递Header`X-Forwarded-Proto：https` ，可以禁用此重定向。

The load balancer or proxy has to be configured to support the following:
负载平衡器或代理必须配置为支持以下各项：

- **WebSocket** connections
- WebSocket连接
- **SPDY** / **HTTP/2** protocols
- SPDY / HTTP/2 协议
- Passing / setting the following headers:
- 传递以下headers：

  | Header              | Value                           | Description                                                                                                                                                                             |
  | ------------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | `Host`              | Hostname used to reach Rancher. | To identify the server requested by the client.                                                                                                                                         |
  | `X-Forwarded-Proto` | `https`                         | To identify the protocol that a client used to connect to the load balancer or proxy.<br /><br/>**Note:** If this header is present, `rancher/rancher` does not redirect HTTP to HTTPS. |
  | `X-Forwarded-Port`  | Port used to reach Rancher.     | To identify the protocol that client used to connect to the load balancer or proxy.                                                                                                     |
  | `X-Forwarded-For`   | IP of the client connection.    | To identify the originating IP address of a client.                                                                                                                                     |

  | Header              | Value                           | Description                                                                                                                                                                             |
  | ------------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | `Host`              | 可达Rancher的 hostname |  用来表示接受请求的Rancher server                                                                                                                                        |
  | `X-Forwarded-Proto` | `https`                         |  标识客户端用来连接到负载均衡器或代理的协议。<br /> <br/> **注意：**如果存在此标头，则`rancher / rancher`不会将HTTP重定向到HTTPS。|
  | `X-Forwarded-Port`  | 可达Rancher的端口     | 标识客户端用来连接到负载均衡器或代理的协议。                                                                                                    |
  | `X-Forwarded-For`   | 请求端IP地址    | 用来表示请求端原始IP                                                                                                                                     |

#### Example Nginx configuration

This NGINX configuration is tested on NGINX 1.14.
此NGINX配置已在NGINX 1.14上进行了测试。

> **Note:** This Nginx configuration is only an example and may not suit your environment. For complete documentation, see [NGINX Load Balancing - HTTP Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/).
> **注意：**此Nginx配置仅是示例，可能不适合您的环境。 有关完整的文档，请参阅[NGINX负载平衡-HTTP负载平衡](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)。

- Replace `rancher-server` with the IP address or hostname of the node running the Rancher container.
- 将“ rancher-server”替换为运行Rancher容器的节点的IP地址或主机名。
- Replace both occurrences of `FQDN` to the DNS name for Rancher.
- 将两次出现的“ FQDN”替换为Rancher的DNS名称。
- Replace `/certs/fullchain.pem` and `/certs/privkey.pem` to the location of the server certificate and the server certificate key respectively.
-将“ /certs/fullchain.pem”和“ /certs/privkey.pem”分别替换为服务器证书和服务器证书密钥的位置。

```
worker_processes 4;
worker_rlimit_nofile 40000;

events {
    worker_connections 8192;
}

http {
    upstream rancher {
        server rancher-server:80;
    }

    map $http_upgrade $connection_upgrade {
        default Upgrade;
        ''      close;
    }

    server {
        listen 443 ssl http2;
        server_name FQDN;
        ssl_certificate /certs/fullchain.pem;
        ssl_certificate_key /certs/privkey.pem;

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Port $server_port;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://rancher;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
            # This allows the ability for the execute shell window to remain open for up to 15 minutes. Without this parameter, the default is 1 minute and will automatically close.
            proxy_read_timeout 900s;
            proxy_buffering off;
        }
    }

    server {
        listen 80;
        server_name FQDN;
        return 301 https://$server_name$request_uri;
    }
}
```

<br/>

### What's Next?
还有什么？

- **Recommended:** Review [Single Node Backup and Restoration](/docs/installation/backups-and-restoration/single-node-backup-and-restoration/). Although you don't have any data you need to back up right now, we recommend creating backups after regular Rancher use.
- 推荐阅读：[单节点备份和还原](/docs/installation/backups-and-restoration/single-node-backup-and-restoration/). 尽管您现在没有任何数据需要备份，但是我们建议您在常规Rancher使用之后创建备份
- Create a Kubernetes cluster: [Provisioning Kubernetes Clusters](/docs/cluster-provisioning/).
- 创建Kubernetes集群[Provisioning Kubernetes Clusters](/docs/cluster-provisioning/)

<br/>

### FAQ and Troubleshooting

{{< ssl_faq_single >}}

### Advanced Options

#### API Auditing
API审计

If you want to record all transactions with the Rancher API, enable the [API Auditing](/docs/installation/api-auditing) feature by adding the flags below into your install command.
如果要使用Rancher API记录所有事务，请通过在安装命令中添加以下标志来启用[API审计](/docs/installation/api-auditing)功能。

    -e AUDIT_LEVEL=1 \
    -e AUDIT_LOG_PATH=/var/log/auditlog/rancher-api-audit.log \
    -e AUDIT_LOG_MAXAGE=20 \
    -e AUDIT_LOG_MAXBACKUP=20 \
    -e AUDIT_LOG_MAXSIZE=100 \

#### Air Gap

If you are visiting this page to complete an [Air Gap Installation](/docs/installation/air-gap-installation/), you must pre-pend your private registry URL to the server tag when running the installation command in the option that you choose. Add `<REGISTRY.DOMAIN.COM:PORT>` with your private registry URL in front of `rancher/rancher:latest`.
如果您正在访问此页面以完成[气隙安装](/docs/installation/air-gap-installation/)，则在运行安装命令时，必须在选项中将私有镜像库URL预先附加到server镜像上。 在您的rancher/rancher:latest前面添加带有您的私有镜像库URL的<REGISTRY.DOMAIN.COM:PORT>。

**Example:**

     <REGISTRY.DOMAIN.COM:PORT>/rancher/rancher:latest

#### Persistent Data
持久化数据

{{< persistentdata >}}

This layer 7 Nginx configuration is tested on Nginx version 1.13 (mainline) and 1.14 (stable).
7层Nginx配置已在Nginx版本1.13（主线）和1.14（稳定）上进行了测试。

> **Note:** This Nginx configuration is only an example and may not suit your environment. For complete documentation, see [NGINX Load Balancing - TCP and UDP Load Balancer](https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/).
> **注意：**此Nginx配置仅是示例，可能不适合您的环境。 有关完整的文档，请参阅[NGINX负载平衡-TCP和UDP负载平衡器](https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/)。

```
upstream rancher {
    server rancher-server:80;
}

map $http_upgrade $connection_upgrade {
    default Upgrade;
    ''      close;
}

server {
    listen 443 ssl http2;
    server_name rancher.yourdomain.com;
    ssl_certificate /etc/your_certificate_directory/fullchain.pem;
    ssl_certificate_key /etc/your_certificate_directory/privkey.pem;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://rancher;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        # This allows the ability for the execute shell window to remain open for up to 15 minutes. Without this parameter, the default is 1 minute and will automatically close.
        proxy_read_timeout 900s;
        proxy_buffering off;
    }
}

server {
    listen 80;
    server_name rancher.yourdomain.com;
    return 301 https://$server_name$request_uri;
}
```

<br/>
