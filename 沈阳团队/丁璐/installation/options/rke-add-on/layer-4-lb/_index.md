---
title: 使用外部负载均衡器(TCP/四层)安装Kubernetes
---

> #### **重要提示：Rancher v2.0.8之前仅支持RKE add-on安装**
>
> 请使用Rancher helm chart将Rancher安装在Kubernetes集群上。有关详细信息，请参见[Kubernetes安装-安装概述](/docs/installation/k8s-install/#installation-outline)。
>
> 如果您当前正在使用RKE add-on安装方法，参见[从带有RKE add-on组件的Kubernetes安装迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/)获取有关如何使用Helm chart的详细信息。

此过程将引导您使用Rancher Kubernetes引擎（RKE）设置3节点集群。该集群的唯一目的是为Rancher运行Pod。设置基于：

- 四层负载均衡器(TCP)
- [具有SSL termination(HTTPS)的NGINX ingress控制器](https://kubernetes.github.io/ingress-nginx/)

在使用四层负载均衡器的HA设置中，负载均衡器通过TCP/UDP协议(即，传输级别)接受Rancher客户端连接，然后，负载均衡器将这些连接转发到各个集群节点，而不读取请求本身。由于负载均衡器无法读取其转发的数据包，因此它所能做出的路由决策是有限的。

<sup>Rancher安装在具有四层负载均衡器的Kubernetes集群上，描述了在ingress控制器上的SSL termination。</sup>
![Rancher HA](/img/rancher/ha/rancher2ha.svg)

### 安装概述

在高可用性配置中安装Rancher涉及多个过程，查看此概述以了解您需要完成的每个过程。

<!-- TOC -->

- [1. 提供Linux主机](#1-provision-linux-hosts)
- [2. 配置负载均衡器](#2-configure-load-balancer)
- [3. 配置DNS](#3-configure-dns)
- [4. 安装RKE](#4-install-rke)
- [5. 下载RKE配置文件模板](#5-download-rke-config-file-template)
- [6. 配置节点](#6-configure-nodes)
- [7. 配置证书](#7-configure-certificates)
- [8. 配置FQDN](#8-configure-fqdn)
- [9. 配置Rancher版本](#9-configure-rancher-version)
- [10. 备份RKE配置文件](#10-back-up-your-rke-config-file)
- [11. 运行RKE](#11-run-rke)
- [12. 备份自动生成的配置文件](#12-back-up-auto-generated-config-file)

<!-- /TOC -->

<br/>

### 1. 提供Linux主机

根据我们的[要求](/docs/installation/requirements)配置三台Linux主机。

### 2. 配置负载均衡器

我们将使用NGINX作为我们的四层负载均衡器(TCP)，NGINX将所有连接转发到您的Rancher节点之一。如果要使用Amazon NLB，则可以跳过此步骤并使用[Amazon NLB配置](/docs/installation/options/rke-add-on/layer-4-lb/nlb/)。

> **注意：**
> 在此配置中，负载均衡器位于Linux主机的前面，负载均衡器可以是任何可用的能够运行NGINX的主机。
>
> 一个警告：不要将Rancher节点之一用作负载均衡器。

#### A. 安装NGINX

首先在负载均衡器主机上安装NGINX，NGINX为所有已知的操作系统提供了可用的安装包。有关安装NGINX的帮助，请参阅[安装文档](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)。

在使用官方NGINX软件包时，`stream`模块是必需的。请参考您的操作系统文档，了解如何在操作系统上安装和启用NGINX `stream`模块。

#### B. 创建NGINX配置

安装NGINX后，您需要使用节点的IP地址更新NGINX配置文件`nginx.conf`。

1. 将下面的代码示例复制并粘贴到您喜欢的文本编辑器中，另存为`nginx.conf`。

2. 将`nginx.conf`中的`IP_NODE_1`，`IP_NODE_2`和`IP_NODE_3`替换为您的[Linux主机](#1-provision-linux-hosts)IP地址。

   > **注意：** 此Nginx配置仅是示例，可能不适合您的环境。有关完整的文档，请参阅[NGINX负载平衡-TCP和UDP负载均衡器](https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/)。

   **NGINX配置示例：**

   ```
   worker_processes 4;
   worker_rlimit_nofile 40000;

   events {
       worker_connections 8192;
   }

   http {
       server {
           listen         80;
           return 301 https://$host$request_uri;
       }
   }

   stream {
       upstream rancher_servers {
           least_conn;
           server IP_NODE_1:443 max_fails=3 fail_timeout=5s;
           server IP_NODE_2:443 max_fails=3 fail_timeout=5s;
           server IP_NODE_3:443 max_fails=3 fail_timeout=5s;
       }
       server {
           listen     443;
           proxy_pass rancher_servers;
       }
   }
   ```

3. 保存`nginx.conf`到负载均衡器的以下路径：`/etc/nginx/nginx.conf`。

4. 通过运行以下命令更新加载您的NGINX配置：

   ```
   # nginx -s reload
   ```

#### 可选 - 将NGINX作为Docker容器运行

与其将NGINX作为包安装在操作系统上，不如将其作为Docker容器运行。将已编辑的**NGINX配置示例**保存为`/etc/nginx.conf`，并运行以下命令以启动NGINX容器：

```
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /etc/nginx.conf:/etc/nginx/nginx.conf \
  nginx:1.14
```

### 3. 配置DNS

选择要用于访问Rancher的完全限定域名(FQDN)(例如，`rancher.yourdomain.com`)。<br/><br/>

1. 登录到DNS服务器，创建一个指向您的[负载均衡器]的IP地址的`DNS A`记录(#2-configure-load-balancer)。

2. 验证`DNS A`是否正常工作，在任何终端上运行以下命令，替换`HOSTNAME.DOMAIN.COM`为您选择的FQDN：

   `nslookup HOSTNAME.DOMAIN.COM`

   **步骤结果：** 终端显示输出如下图所示：

   ```
   $ nslookup rancher.yourdomain.com
   Server:         YOUR_HOSTNAME_IP_ADDRESS
   Address:        YOUR_HOSTNAME_IP_ADDRESS#53

   Non-authoritative answer:
   Name:   rancher.yourdomain.com
   Address: HOSTNAME.DOMAIN.COM
   ```

<br/>

### 4. 安装RKE

RKE(Rancher Kubernetes引擎)是一个快速、通用的Kubernetes安装程序，您可以使用它在您的Linux主机上安装Kubernetes。我们将使用RKE来设置集群并运行Rancher。

1. 请遵循[RKE安装]({{<baseurl>}}/rke/latest/en/installation)说明。

2. 通过运行以下命令，确认RKE现在是可执行的：

   ```
   rke --version
   ```

### 5. 下载RKE配置文件模板

RKE使用`.yml`配置文件来安装和配置Kubernetes集群。根据要使用的SSL证书，有两种模板可供选择。

1. 根据您正在使用的SSL证书，下载以下模版之一。

   - [自签名证书模版<br/> `3-node-certificate.yml`](https://raw.githubusercontent.com/rancher/rancher/master/rke-templates/3-node-certificate.yml)
   - [由公认的CA签署的证书模板<br/> `3-node-certificate-recognizedca.yml`](https://raw.githubusercontent.com/rancher/rancher/master/rke-templates/3-node-certificate-recognizedca.yml)

   > **高级配置选项：**
   >
   > - 想要Rancher API的所有事务记录？通过编辑RKE配置文件来启用[API审核](/docs/installation/api-auditing)功能。有关更多信息，请参见如何在[RKE配置文件](/docs/installation/k8s-install/rke-add-on/api-auditing/)中启用它。
   > - 想知道您的RKE模板可用的其他配置选项吗？请参阅[RKE文档：配置选项]({{<baseurl>}}/rke/latest/en/config-options/).

2) 将文件重命名为`rancher-cluster.yml`。

### 6. 配置节点

有了`rancher-cluster.yml`配置文件模板后，编辑节点部分以指向您的Linux主机。

1.  在您喜欢的文本编辑器中打开`rancher-cluster.yml`。

1.  使用[Linux主机](#1-provision-linux-hosts)信息更新`nodes`部分

    对于集群中的每个节点，更新以下占位符：`IP_ADDRESS_X`和`USER`。指定的用户应该能够访问Docket套接字，您可以使用指定的用户登录并运行`docker ps`来测试这一点。

    > **注意：**
    > 使用RHEL/CentOS时，由于https://bugzilla.redhat.com/show_bug.cgi?id=1527565导致SSH用户无法成为root用户。有关RHEL/CentOS的特定需求，请参阅[操作系统要求]({{<baseurl>}}/rke/latest/en/installation/os#redhat-enterprise-linux-rhel-centos)。

        nodes:
            # The IP address or hostname of the node
        - address: IP_ADDRESS_1
            # User that can login to the node and has access to the Docker socket (i.e. can execute `docker ps` on the node)
            # When using RHEL/CentOS, this can't be root due to https://bugzilla.redhat.com/show_bug.cgi?id=1527565
            user: USER
            role: [controlplane,etcd,worker]
            # Path the SSH key that can be used to access to node with the specified user
            ssh_key_path: ~/.ssh/id_rsa
        - address: IP_ADDRESS_2
            user: USER
            role: [controlplane,etcd,worker]
            ssh_key_path: ~/.ssh/id_rsa
        - address: IP_ADDRESS_3
            user: USER
            role: [controlplane,etcd,worker]
            ssh_key_path: ~/.ssh/id_rsa

1.  **可选：** 默认情况下,`rancher-cluster.yml`被配置为备份您的数据快照，要禁用这些快照，请将`backup`指令设置更改为`false`，如下所示。

        services:
          etcd:
            backup: false

### 7. 配置证书

为了安全起见，使用Rancher时需要SSL(Secure Sockets Layer)。SSL保护所有Rancher网络通信的安全，例如在您登录集群或与集群交互时。

从以下选项中选择：

 accordion id="option-a" label="选项A—携带您自己的证书：自签名"

> **先决条件：**
> 创建一个自签名证书
>
> - 证书文件必须为[PEM格式](#pem)。
> - 证书文件必须使用[base64](#base64)编码。
> - 在您的证书文件中，包括链接中的所有中间证书。请先使用证书订购证书，然后再订购中间体。有关示例，请参见[中间证书](#cert-order)。

1. 在`kind: Secret`中`name: cattle-keys-ingress`：

   - 替换`<BASE64_CRT>`为证书文件的base64编码字符串(通常称为`cert.pem`或`domain.crt`)
   - 替换`<BASE64_KEY>`为证书密钥文件的base64编码字符串(通常称为`key.pem`或`domain.key`)

   > **注意：**
   > `tls.crt`或`tls.key`的base64编码的字符串应该在同一行，在开头、中间或结尾没有任何换行。

   **步骤结果：** 在替换了这些值之后，文件应该如下面的示例所示(base64编码的字符串应该不同)：

   ```yaml
   ---
   apiVersion: v1
   kind: Secret
   metadata:
     name: cattle-keys-ingress
     namespace: cattle-system
   type: Opaque
   data:
     tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1RENDQWN5Z0F3SUJBZ0lKQUlHc25NeG1LeGxLTUEwR0NTcUdTSWIzRFFFQkN3VUFNQkl4RURBT0JnTlYKQkFNTUIzUmxjM1F0WTJFd0hoY05NVGd3TlRBMk1qRXdOREE1V2hjTk1UZ3dOekExTWpFd05EQTVXakFXTVJRdwpFZ1lEVlFRRERBdG9ZUzV5Ym1Ob2NpNXViRENDQVNJd0RRWUpLb1pJaHZjTkFRRUJCUUFEZ2dFUEFEQ0NBUW9DCmdnRUJBTFJlMXdzekZSb2Rib2pZV05DSHA3UkdJaUVIMENDZ1F2MmdMRXNkUUNKZlcrUFEvVjM0NnQ3bSs3TFEKZXJaV3ZZMWpuY2VuWU5JSGRBU0VnU0ducWExYnhUSU9FaE0zQXpib3B0WDhjSW1OSGZoQlZETGdiTEYzUk0xaQpPM1JLTGdIS2tYSTMxZndjbU9zWGUwaElYQnpUbmxnM20vUzlXL3NTc0l1dDVwNENDUWV3TWlpWFhuUElKb21lCmpkS3VjSHFnMTlzd0YvcGVUalZrcVpuMkJHazZRaWFpMU41bldRV0pjcThTenZxTTViZElDaWlwYU9hWWQ3RFEKYWRTejV5dlF0YkxQNW4wTXpnOU43S3pGcEpvUys5QWdkWDI5cmZqV2JSekp3RzM5R3dRemN6VWtLcnZEb05JaQo0UFJHc01yclFNVXFSYjRSajNQOEJodEMxWXNDQXdFQUFhTTVNRGN3Q1FZRFZSMFRCQUl3QURBTEJnTlZIUThFCkJBTUNCZUF3SFFZRFZSMGxCQll3RkFZSUt3WUJCUVVIQXdJR0NDc0dBUVVGQndNQk1BMEdDU3FHU0liM0RRRUIKQ3dVQUE0SUJBUUNKZm5PWlFLWkowTFliOGNWUW5Vdi9NZkRZVEJIQ0pZcGM4MmgzUGlXWElMQk1jWDhQRC93MgpoOUExNkE4NGNxODJuQXEvaFZYYy9JNG9yaFY5WW9jSEg5UlcvbGthTUQ2VEJVR0Q1U1k4S292MHpHQ1ROaDZ6Ci9wZTNqTC9uU0pYSjRtQm51czJheHFtWnIvM3hhaWpYZG9kMmd3eGVhTklvRjNLbHB2aGU3ZjRBNmpsQTM0MmkKVVlCZ09iN1F5KytRZWd4U1diSmdoSzg1MmUvUUhnU2FVSkN6NW1sNGc1WndnNnBTUXhySUhCNkcvREc4dElSYwprZDMxSk1qY25Fb1Rhc1Jyc1NwVmNGdXZyQXlXN2liakZyYzhienBNcE1obDVwYUZRcEZzMnIwaXpZekhwakFsCk5ZR2I2OHJHcjBwQkp3YU5DS2ErbCtLRTk4M3A3NDYwCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
     tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBdEY3WEN6TVZHaDF1aU5oWTBJZW50RVlpSVFmUUlLQkMvYUFzU3gxQUlsOWI0OUQ5ClhmanEzdWI3c3RCNnRsYTlqV09keDZkZzBnZDBCSVNCSWFlcHJWdkZNZzRTRXpjRE51aW0xZnh3aVkwZCtFRlUKTXVCc3NYZEV6V0k3ZEVvdUFjcVJjamZWL0J5WTZ4ZDdTRWhjSE5PZVdEZWI5TDFiK3hLd2k2M21uZ0lKQjdBeQpLSmRlYzhnbWlaNk4wcTV3ZXFEWDJ6QVgrbDVPTldTcG1mWUVhVHBDSnFMVTNtZFpCWWx5cnhMTytvemx0MGdLCktLbG81cGgzc05CcDFMUG5LOUMxc3MvbWZRek9EMDNzck1Xa21oTDcwQ0IxZmIydCtOWnRITW5BYmYwYkJETnoKTlNRcXU4T2cwaUxnOUVhd3l1dEF4U3BGdmhHUGMvd0dHMExWaXdJREFRQUJBb0lCQUJKYUErOHp4MVhjNEw0egpwUFd5bDdHVDRTMFRLbTNuWUdtRnZudjJBZXg5WDFBU2wzVFVPckZyTnZpK2xYMnYzYUZoSFZDUEN4N1RlMDVxClhPa2JzZnZkZG5iZFQ2RjgyMnJleVByRXNINk9TUnBWSzBmeDVaMDQwVnRFUDJCWm04eTYyNG1QZk1vbDdya2MKcm9Kd09rOEVpUHZZekpsZUd0bTAwUm1sRysyL2c0aWJsOTVmQXpyc1MvcGUyS3ZoN2NBVEtIcVh6MjlpUmZpbApiTGhBamQwcEVSMjNYU0hHR1ZqRmF3amNJK1c2L2RtbDZURDhrSzFGaUtldmJKTlREeVNXQnpPbXRTYUp1K01JCm9iUnVWWG4yZVNoamVGM1BYcHZRMWRhNXdBa0dJQWxOWjRHTG5QU2ZwVmJyU0plU3RrTGNzdEJheVlJS3BWZVgKSVVTTHM0RUNnWUVBMmNnZUE2WHh0TXdFNU5QWlNWdGhzbXRiYi9YYmtsSTdrWHlsdk5zZjFPdXRYVzkybVJneQpHcEhUQ0VubDB0Z1p3T081T1FLNjdFT3JUdDBRWStxMDJzZndwcmgwNFZEVGZhcW5QNTBxa3BmZEJLQWpmanEyCjFoZDZMd2hLeDRxSm9aelp2VkowV0lvR1ZLcjhJSjJOWGRTUVlUanZUZHhGczRTamdqNFFiaEVDZ1lFQTFBWUUKSEo3eVlza2EvS2V2OVVYbmVrSTRvMm5aYjJ1UVZXazRXSHlaY2NRN3VMQVhGY3lJcW5SZnoxczVzN3RMTzJCagozTFZNUVBzazFNY25oTTl4WE4vQ3ZDTys5b2t0RnNaMGJqWFh6NEJ5V2lFNHJPS1lhVEFwcDVsWlpUT3ZVMWNyCm05R3NwMWJoVDVZb2RaZ3IwUHQyYzR4U2krUVlEWnNFb2lFdzNkc0NnWUVBcVJLYWNweWZKSXlMZEJjZ0JycGkKQTRFalVLMWZsSjR3enNjbGFKUDVoM1NjZUFCejQzRU1YT0kvSXAwMFJsY3N6em83N3cyMmpud09mOEJSM0RBMwp6ZTRSWDIydWw4b0hGdldvdUZOTTNOZjNaNExuYXpVc0F0UGhNS2hRWGMrcEFBWGthUDJkZzZ0TU5PazFxaUNHCndvU212a1BVVE84b1ViRTB1NFZ4ZmZFQ2dZQUpPdDNROVNadUlIMFpSSitIV095enlOQTRaUEkvUkhwN0RXS1QKajVFS2Y5VnR1OVMxY1RyOTJLVVhITXlOUTNrSjg2OUZPMnMvWk85OGg5THptQ2hDTjhkOWN6enI5SnJPNUFMTApqWEtBcVFIUlpLTFgrK0ZRcXZVVlE3cTlpaHQyMEZPb3E5OE5SZDMzSGYxUzZUWDNHZ3RWQ21YSml6dDAxQ3ZHCmR4VnVnd0tCZ0M2Mlp0b0RLb3JyT2hvdTBPelprK2YwQS9rNDJBOENiL29VMGpwSzZtdmxEWmNYdUF1QVZTVXIKNXJCZjRVYmdVYndqa1ZWSFR6LzdDb1BWSjUvVUxJWk1Db1RUNFprNTZXWDk4ZE93Q3VTVFpZYnlBbDZNS1BBZApTZEpuVVIraEpnSVFDVGJ4K1dzYnh2d0FkbWErWUhtaVlPRzZhSklXMXdSd1VGOURLUEhHCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
   ```

2. 在`kind: Secret`中`name: cattle-keys-server`，替换`<BASE64_CA>`为CA证书文件的base64编码的字符串(通常称为`ca.pem`或`ca.crt`)。

   > **注意：**
   > `cacerts.pem`的base64编码的字符串应该在同一行，在开头、中间或结尾没有任何换行。

    **步骤结果：** 文件应该如下面的示例所示(base64编码的字符串应该不同)：

    ```yaml
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: cattle-keys-server
      namespace: cattle-system
    type: Opaque
    data:
      cacerts.pem: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNvRENDQVlnQ0NRRHVVWjZuMEZWeU16QU5CZ2txaGtpRzl3MEJBUXNGQURBU01SQXdEZ1lEVlFRRERBZDAKWlhOMExXTmhNQjRYRFRFNE1EVXdOakl4TURRd09Wb1hEVEU0TURjd05USXhNRFF3T1Zvd0VqRVFNQTRHQTFVRQpBd3dIZEdWemRDMWpZVENDQVNJd0RRWUpLb1pJaHZjTkFRRUJCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFNQmpBS3dQCndhRUhwQTdaRW1iWWczaTNYNlppVmtGZFJGckJlTmFYTHFPL2R0RUdmWktqYUF0Wm45R1VsckQxZUlUS3UzVHgKOWlGVlV4Mmo1Z0tyWmpwWitCUnFiZ1BNbk5hS1hocmRTdDRtUUN0VFFZdGRYMVFZS0pUbWF5NU45N3FoNTZtWQprMllKRkpOWVhHWlJabkdMUXJQNk04VHZramF0ZnZOdmJ0WmtkY2orYlY3aWhXanp2d2theHRUVjZlUGxuM2p5CnJUeXBBTDliYnlVcHlad3E2MWQvb0Q4VUtwZ2lZM1dOWmN1YnNvSjhxWlRsTnN6UjVadEFJV0tjSE5ZbE93d2oKaG41RE1tSFpwZ0ZGNW14TU52akxPRUc0S0ZRU3laYlV2QzlZRUhLZTUxbGVxa1lmQmtBZWpPY002TnlWQUh1dApuay9DMHpXcGdENkIwbkVDQXdFQUFUQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFHTCtaNkRzK2R4WTZsU2VBClZHSkMvdzE1bHJ2ZXdia1YxN3hvcmlyNEMxVURJSXB6YXdCdFJRSGdSWXVtblVqOGo4T0hFWUFDUEthR3BTVUsKRDVuVWdzV0pMUUV0TDA2eTh6M3A0MDBrSlZFZW9xZlVnYjQrK1JLRVJrWmowWXR3NEN0WHhwOVMzVkd4NmNOQQozZVlqRnRQd2hoYWVEQmdma1hXQWtISXFDcEsrN3RYem9pRGpXbi8walI2VDcrSGlaNEZjZ1AzYnd3K3NjUDIyCjlDQVZ1ZFg4TWpEQ1hTcll0Y0ZINllBanlCSTJjbDhoSkJqa2E3aERpVC9DaFlEZlFFVFZDM3crQjBDYjF1NWcKdE03Z2NGcUw4OVdhMnp5UzdNdXk5bEthUDBvTXl1Ty82Tm1wNjNsVnRHeEZKSFh4WTN6M0lycGxlbTNZQThpTwpmbmlYZXc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    ```

 /accordion 

 accordion id="option-b" label="选项B—携带您自己的证书：由公认的CA签署"

> **注意：**
> 如果您使用的是自签名证书，[单击此处](#option-a-bring-your-own-certificate-self-signed)继续。

如果您使用的是由公认的证书颁发机构签名的证书，您需要为证书文件和证书密钥文件生成base64编码的字符串。确保您的证书文件包括链接中的所有[中间证书](#cert-order)，在这种情况下，证书的顺序首先是您自己的证书，然后是中间证书。请参阅您的CSP(Certificate Service Provider)文档，了解需要包括哪些中间证书。

在`kind: Secret`中`name: cattle-keys-ingress`：

- 替换`<BASE64_CRT>`为证书文件的base64编码字符串(通常称为`cert.pem`或`domain.crt`)
- 替换`<BASE64_KEY>`为证书密钥文件的base64编码字符串(通常称为`key.pem`或`domain.key`)

在替换了这些值之后，文件应该如下面的示例所示(base64编码的字符串应该不同)：

> **注意：**
> `tls.crt`或`tls.key`的base64编码的字符串应该在同一行，在开头、中间或结尾没有任何换行。

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: cattle-keys-ingress
  namespace: cattle-system
type: Opaque
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1RENDQWN5Z0F3SUJBZ0lKQUlHc25NeG1LeGxLTUEwR0NTcUdTSWIzRFFFQkN3VUFNQkl4RURBT0JnTlYKQkFNTUIzUmxjM1F0WTJFd0hoY05NVGd3TlRBMk1qRXdOREE1V2hjTk1UZ3dOekExTWpFd05EQTVXakFXTVJRdwpFZ1lEVlFRRERBdG9ZUzV5Ym1Ob2NpNXViRENDQVNJd0RRWUpLb1pJaHZjTkFRRUJCUUFEZ2dFUEFEQ0NBUW9DCmdnRUJBTFJlMXdzekZSb2Rib2pZV05DSHA3UkdJaUVIMENDZ1F2MmdMRXNkUUNKZlcrUFEvVjM0NnQ3bSs3TFEKZXJaV3ZZMWpuY2VuWU5JSGRBU0VnU0ducWExYnhUSU9FaE0zQXpib3B0WDhjSW1OSGZoQlZETGdiTEYzUk0xaQpPM1JLTGdIS2tYSTMxZndjbU9zWGUwaElYQnpUbmxnM20vUzlXL3NTc0l1dDVwNENDUWV3TWlpWFhuUElKb21lCmpkS3VjSHFnMTlzd0YvcGVUalZrcVpuMkJHazZRaWFpMU41bldRV0pjcThTenZxTTViZElDaWlwYU9hWWQ3RFEKYWRTejV5dlF0YkxQNW4wTXpnOU43S3pGcEpvUys5QWdkWDI5cmZqV2JSekp3RzM5R3dRemN6VWtLcnZEb05JaQo0UFJHc01yclFNVXFSYjRSajNQOEJodEMxWXNDQXdFQUFhTTVNRGN3Q1FZRFZSMFRCQUl3QURBTEJnTlZIUThFCkJBTUNCZUF3SFFZRFZSMGxCQll3RkFZSUt3WUJCUVVIQXdJR0NDc0dBUVVGQndNQk1BMEdDU3FHU0liM0RRRUIKQ3dVQUE0SUJBUUNKZm5PWlFLWkowTFliOGNWUW5Vdi9NZkRZVEJIQ0pZcGM4MmgzUGlXWElMQk1jWDhQRC93MgpoOUExNkE4NGNxODJuQXEvaFZYYy9JNG9yaFY5WW9jSEg5UlcvbGthTUQ2VEJVR0Q1U1k4S292MHpHQ1ROaDZ6Ci9wZTNqTC9uU0pYSjRtQm51czJheHFtWnIvM3hhaWpYZG9kMmd3eGVhTklvRjNLbHB2aGU3ZjRBNmpsQTM0MmkKVVlCZ09iN1F5KytRZWd4U1diSmdoSzg1MmUvUUhnU2FVSkN6NW1sNGc1WndnNnBTUXhySUhCNkcvREc4dElSYwprZDMxSk1qY25Fb1Rhc1Jyc1NwVmNGdXZyQXlXN2liakZyYzhienBNcE1obDVwYUZRcEZzMnIwaXpZekhwakFsCk5ZR2I2OHJHcjBwQkp3YU5DS2ErbCtLRTk4M3A3NDYwCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  tls.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBdEY3WEN6TVZHaDF1aU5oWTBJZW50RVlpSVFmUUlLQkMvYUFzU3gxQUlsOWI0OUQ5ClhmanEzdWI3c3RCNnRsYTlqV09keDZkZzBnZDBCSVNCSWFlcHJWdkZNZzRTRXpjRE51aW0xZnh3aVkwZCtFRlUKTXVCc3NYZEV6V0k3ZEVvdUFjcVJjamZWL0J5WTZ4ZDdTRWhjSE5PZVdEZWI5TDFiK3hLd2k2M21uZ0lKQjdBeQpLSmRlYzhnbWlaNk4wcTV3ZXFEWDJ6QVgrbDVPTldTcG1mWUVhVHBDSnFMVTNtZFpCWWx5cnhMTytvemx0MGdLCktLbG81cGgzc05CcDFMUG5LOUMxc3MvbWZRek9EMDNzck1Xa21oTDcwQ0IxZmIydCtOWnRITW5BYmYwYkJETnoKTlNRcXU4T2cwaUxnOUVhd3l1dEF4U3BGdmhHUGMvd0dHMExWaXdJREFRQUJBb0lCQUJKYUErOHp4MVhjNEw0egpwUFd5bDdHVDRTMFRLbTNuWUdtRnZudjJBZXg5WDFBU2wzVFVPckZyTnZpK2xYMnYzYUZoSFZDUEN4N1RlMDVxClhPa2JzZnZkZG5iZFQ2RjgyMnJleVByRXNINk9TUnBWSzBmeDVaMDQwVnRFUDJCWm04eTYyNG1QZk1vbDdya2MKcm9Kd09rOEVpUHZZekpsZUd0bTAwUm1sRysyL2c0aWJsOTVmQXpyc1MvcGUyS3ZoN2NBVEtIcVh6MjlpUmZpbApiTGhBamQwcEVSMjNYU0hHR1ZqRmF3amNJK1c2L2RtbDZURDhrSzFGaUtldmJKTlREeVNXQnpPbXRTYUp1K01JCm9iUnVWWG4yZVNoamVGM1BYcHZRMWRhNXdBa0dJQWxOWjRHTG5QU2ZwVmJyU0plU3RrTGNzdEJheVlJS3BWZVgKSVVTTHM0RUNnWUVBMmNnZUE2WHh0TXdFNU5QWlNWdGhzbXRiYi9YYmtsSTdrWHlsdk5zZjFPdXRYVzkybVJneQpHcEhUQ0VubDB0Z1p3T081T1FLNjdFT3JUdDBRWStxMDJzZndwcmgwNFZEVGZhcW5QNTBxa3BmZEJLQWpmanEyCjFoZDZMd2hLeDRxSm9aelp2VkowV0lvR1ZLcjhJSjJOWGRTUVlUanZUZHhGczRTamdqNFFiaEVDZ1lFQTFBWUUKSEo3eVlza2EvS2V2OVVYbmVrSTRvMm5aYjJ1UVZXazRXSHlaY2NRN3VMQVhGY3lJcW5SZnoxczVzN3RMTzJCagozTFZNUVBzazFNY25oTTl4WE4vQ3ZDTys5b2t0RnNaMGJqWFh6NEJ5V2lFNHJPS1lhVEFwcDVsWlpUT3ZVMWNyCm05R3NwMWJoVDVZb2RaZ3IwUHQyYzR4U2krUVlEWnNFb2lFdzNkc0NnWUVBcVJLYWNweWZKSXlMZEJjZ0JycGkKQTRFalVLMWZsSjR3enNjbGFKUDVoM1NjZUFCejQzRU1YT0kvSXAwMFJsY3N6em83N3cyMmpud09mOEJSM0RBMwp6ZTRSWDIydWw4b0hGdldvdUZOTTNOZjNaNExuYXpVc0F0UGhNS2hRWGMrcEFBWGthUDJkZzZ0TU5PazFxaUNHCndvU212a1BVVE84b1ViRTB1NFZ4ZmZFQ2dZQUpPdDNROVNadUlIMFpSSitIV095enlOQTRaUEkvUkhwN0RXS1QKajVFS2Y5VnR1OVMxY1RyOTJLVVhITXlOUTNrSjg2OUZPMnMvWk85OGg5THptQ2hDTjhkOWN6enI5SnJPNUFMTApqWEtBcVFIUlpLTFgrK0ZRcXZVVlE3cTlpaHQyMEZPb3E5OE5SZDMzSGYxUzZUWDNHZ3RWQ21YSml6dDAxQ3ZHCmR4VnVnd0tCZ0M2Mlp0b0RLb3JyT2hvdTBPelprK2YwQS9rNDJBOENiL29VMGpwSzZtdmxEWmNYdUF1QVZTVXIKNXJCZjRVYmdVYndqa1ZWSFR6LzdDb1BWSjUvVUxJWk1Db1RUNFprNTZXWDk4ZE93Q3VTVFpZYnlBbDZNS1BBZApTZEpuVVIraEpnSVFDVGJ4K1dzYnh2d0FkbWErWUhtaVlPRzZhSklXMXdSd1VGOURLUEhHCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
```

 /accordion 

### 8. 配置FQDN

`<FQDN>`在配置文件中有两处引用(一个在这个步骤中，一个在下一个步骤中)。两者都需要替换为[配置DNS](#3-configure-dns)中选择的FQDN。

在`kind: Ingress`中`name: cattle-ingress-http`：

- 替换`<FQDN>`为[配置DNS](#3-configure-dns)中选择的FQDN。

将`<FQDN>`替换为[配置DNS](#3-configure-dns)中选择的FQDN后，该文件应类似于以下示例(本例中使用的FQDN是`rancher.yourdomain.com`)：

```yaml
 ---
  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    namespace: cattle-system
    name: cattle-ingress-http
    annotations:
      nginx.ingress.kubernetes.io/proxy-connect-timeout: "30"
      nginx.ingress.kubernetes.io/proxy-read-timeout: "1800"   # Max time in seconds for ws to remain shell window open
      nginx.ingress.kubernetes.io/proxy-send-timeout: "1800"   # Max time in seconds for ws to remain shell window open
  spec:
    rules:
    - host: rancher.yourdomain.com
      http:
        paths:
        - backend:
            serviceName: cattle-service
            servicePort: 80
    tls:
    - secretName: cattle-keys-ingress
      hosts:
      - rancher.yourdomain.com
```

保存`.yml`文件并关闭它。

### 9. 配置Rancher版本

最后一个需要替换的引用是`<RANCHER_VERSION>`，这需要替换为标记为稳定的Rancher版本。最新的Rancher稳定版本可以在[GitHub README](https://github.com/rancher/rancher/blob/master/README.md)中找到。确保版本是实际的版本号，而不是带有`stable`或`latest`这样的命名标签。下面的示例显示了配置为`v2.0.6`的版本。

```
      spec:
        serviceAccountName: cattle-admin
        containers:
        - image: rancher/rancher:v2.0.6
          imagePullPolicy: Always
```

### 10. 备份RKE配置文件

关闭`.yml`文件后，将其备份到安全位置。升级Rancher时，可以再次使用此文件。

### 11. 运行RKE

完成所有配置后，使用RKE启动Rancher。您可以通过运行`rke up`命令并使用`--config`参数指向您的配置文件来完成此操作。

1. 在您的工作站中，确保`rancher-cluster.yml`和下载的`rke`二进制文件位于同一目录中。

2. 打开一个终端实例，切换到包含您的配置文件和`rke`的目录。

3. 请输入下面的`rke up`命令。

```
rke up --config rancher-cluster.yml
```

**步骤结果：** 输出应与以下代码段相似：

```
INFO[0000] Building Kubernetes cluster
INFO[0000] [dialer] Setup tunnel for host [1.1.1.1]
INFO[0000] [network] Deploying port listener containers
INFO[0000] [network] Pulling image [alpine:latest] on host [1.1.1.1]
...
INFO[0101] Finished building Kubernetes cluster successfully
```

### 12. 备份自动生成的配置文件

在安装过程中，RKE会自动生成一个与RKE二进制文件位于同一目录中的名为`kube_config_rancher-cluster.yml`的配置文件。复制此文件并将其备份到安全位置，稍后在升级Rancher Server时将使用此文件。

### 下一步是什么？

您有两种选择：

- 在发生灾难的情况下，为您的Rancher Server创建备份：[高可用性备份和恢复](/docs/installation/backups-and-restoration/ha-backup-and-restoration)。
- 创建Kubernetes集群：[提供Kubernetes集群](/docs/cluster-provisioning/)。

<br/>

### 常见问题和故障排除

{{< ssl_faq_ha >}}
