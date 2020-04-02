---
标题: Kubernetes使用外部负载均衡器安装 (HTTPS/Layer 7)
---

> #### **重要说明: Rancher v2.0.8之前仅支持RKE附加安装**
>
> 请使用Rancher Helm chart将Rancher安装在Kubernetes集群上。有关详细信息，请参见[Kubernetes安装-安装概述](/docs/installation/k8s-install/#installation-outline)。
>
> 如果您当前正在使用RKE附加组件安装方法，请参阅[使用RKE附加组件从Kubernetes安装迁移](/docs/upgrades/upgrades/migrating-from-rke-add-on/)以获取有关如何使用Helm chart的详细信息。

此过程将引导您使用Rancher Kubernetes引擎（RKE）设置3-node集群。 该集群的唯一目的是为Rancher运行Pod。设置基于:

- 具有SSL termination（HTTPS）的第7层负载均衡器
- [NGINX入口控制器（HTTP）](https://kubernetes.github.io/ingress-nginx/)

在使用第7层负载均衡器的HA设置中，负载均衡器通过HTTP协议（即应用程序级别）接受Rancher客户端连接。这种应用程序级别的访问允许负载均衡器读取客户端请求，然后使用优化分配负载的逻辑将其重定向到集群节点。

<sup>Rancher安装在具有第7层负载均衡器的Kubernetes集群上，描绘了负载均衡器的SSL termination</sup>
![Rancher HA](/img/rancher/ha/rancher2ha-l7.svg)

### 安装概要

在高可用性配置中安装Rancher涉及多个过程。查看此概述以了解您需要完成的每个过程。

<!-- TOC -->

- [1. 供应Linux主机](#1-provision-linux-hosts)
- [2. 配置负载均衡器](#2-configure-load-balancer)
- [3. 配置DNS](#3-configure-dns)
- [4. 安装RKE](#4-install-rke)
- [5. 下载RKE配置文件模板](#5-download-rke-config-file-template)
- [6. 配置节点](#6-configure-nodes)
- [7. 配置证书](#7-configure-certificates)
- [8. 配置FQDN](#8-configure-fqdn)
- [9. 配置Rancher版本](#9-configure-rancher-version)
- [10. 备份您的RKE配置文件](#10-back-up-your-rke-config-file)
- [11. 运行RKE](#11-run-rke)
- [12. 备份自动生成的配置文件](#12-back-up-auto-generated-config-file)

<!-- /TOC -->

### 1. 供应Linux主机

根据我们的[要求](/docs/installation/requirements)配置三台Linux主机。

### 2. 配置负载均衡器

在Rancher前面使用负载平衡器时，容器无需从端口80或端口443重定向端口通信。通过传递标头`X-Forwarded-Proto: https`，将禁用此重定向。这是在外部终止SSL时的预期配置。

负载平衡器必须配置为支持以下各项：

- **WebSocket** 连接
- **SPDY** / **HTTP/2** 协议
- 传递/设置以下标题：

| 标头                 | 值                           | 描述                                                                                                                                                                            |
| ------------------- | ---------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Host`              | FQDN用于连接Rancher。          | 标识客户端请求的服务器。                                                                                                                                                           |
| `X-Forwarded-Proto` | `https`                      | 标识客户端用来连接到负载均衡器的协议。<br /><br/>**注意:** 如果存在此标头, `rancher/rancher` 则不会将HTTP重定向到HTTPS。                                                                    |
| `X-Forwarded-Port`  | 用来连接Rancher的端口。         | To identify the protocol that client used to connect to the load balancer.                                                                                                     |
| `X-Forwarded-For`   | 客户端连接的IP。                | To identify the originating IP address of a client.                                                                                                                            |

可以在节点的`/healthz`端点上执行健康检查，这将返回HTTP 200。

我们为以下负载均衡器提供了示例配置：

- [Amazon ALB配置](alb/)
- [NGINX配置](nginx/)

### 3. 配置DNS

选择您要用来访问Rancher的标准域名（FQDN）（例如`rancher.yourdomain.com`）。<br/><br/>

1. 登录到DNS服务器，创建一个`DNS A`指向您的[负载平衡器](#2-configure-load-balancer)IP地址的记录。

2. 验证`DNS A`能否正常工作。从任何终端运行以下命令，替换`HOSTNAME.DOMAIN.COM`为您选择的FQDN：

   `nslookup HOSTNAME.DOMAIN.COM`

   **步骤结果:** 终端显示类似于以下内容的输出：

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

RKE（Rancher Kubernetes引擎）是一种快速，通用的Kubernetes安装程序，可用于在Linux主机上安装Kubernetes。我们将使用RKE设置集群并运行Rancher。

1. 请遵循[RKE安装]({{<baseurl>}}/rke/latest/en/installation)说明。

2. 通过运行以下命令，确认RKE现在是可执行的：

   ```
   rke --version
   ```

### 5. 下载RKE配置文件模板

RKE使用YAML配置文件来安装和配置Kubernetes集群。根据要使用的SSL证书，有2种模板可供选择。

1. 下载以下模板之一，具体取决于您使用的SSL证书。

   - [自签名证书的模板<br/> `3-node-externalssl-certificate.yml`](https://raw.githubusercontent.com/rancher/rancher/master/rke-templates/3-node-externalssl-certificate.yml)
   - [由公认的CA签署的证书模板<br/> `3-node-externalssl-recognizedca.yml`](https://raw.githubusercontent.com/rancher/rancher/master/rke-templates/3-node-externalssl-recognizedca.yml)

   > **高级配置选项：**
   >
   > - 想要Rancher API的所有事务记录? 通过编辑RKE配置文件来启用[API审核](/docs/installation/api-auditing)功能。有关更多信息，请参见如何在[RKE配置文件中](/docs/installation/k8s-install/rke-add-on/api-auditing/)中启用它。
   > - 想知道您的RKE模板可用的其他配置选项吗？ 请参阅[RKE文档：配置选项]({{<baseurl>}}/rke/latest/en/config-options/)。

2) 将文件重命名为 `rancher-cluster.yml`.

### 6. 配置节点

有了 `rancher-cluster.yml` 配置文件模板后，编辑节点部分以指向您的Linux主机。

1.  在您喜欢的文本编辑器中打开`rancher-cluster.yml`。

1.  `nodes` 使用[Linux主机](#1-provision-linux-hosts)的信息更新此部分。

    对于集群中的每个节点，更新以下占位符：`IP_ADDRESS_X` 和 `USER`。指定的用户应该能够访问Docket socket，您可以通过使用指定的用户登录并运行来对其进行测试`docker ps`。

    > **注意:**
    >
    > 使用RHEL / CentOS时，由于https://bugzilla.redhat.com/show_bug.cgi?id=1527565导致SSH用户无法成为root用户。有关RHEL / CentOS的特定要求，请参阅[操作系统]({{<baseurl>}}/rke/latest/en/installation/os#redhat-enterprise-linux-rhel-centos)要求。

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

1.  **可选:** 默认情况下, `rancher-cluster.yml` 配置为获取数据的备份快照。要禁用这些快照，请将 `backup` 伪指令设置更改为 `false`，如下所示。

        services:
          etcd:
            backup: false

### 7. 配置证书

为了安全起见，使用Rancher时需要SSL（安全套接字层）。SSL保护所有Rancher网络通信的安全，例如在您登录集群或与集群交互时。

从以下选项中选择：

 accordion id="option-a" label="Option A—Bring Your Own Certificate: Self-Signed" 

> **先决条件:**
> 创建一个自签名证书。
>
> - 证书文件必须为 [PEM 格式](#pem)。
> - 证书文件必须使用 [base64](#base64)编码。
> - 在您的证书文件中，包括链中的所有中间证书。请先使用证书订购证书，然后再订购中间体。有关示例，请参阅 [SSL常见问题/故障排除](#cert-order)。

在 `kind: Secret` 中 `name: cattle-keys-ingress`, 用 `<BASE64_CA>` CA证书文件的base64编码字符串（通常称为 `ca.pem` 或 `ca.crt`）替换。

> **注意:** base64编码的字符串应与相同 `cacerts.pem`，在开头，中间或结尾没有换行符。

替换值之后，文件应类似于以下示例（base64编码的字符串应不同）：

        ---
        apiVersion: v1
        kind: Secret
        metadata:
            name: cattle-keys-server
            namespace: cattle-system
        type: Opaque
        data:
            cacerts.pem: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNvRENDQVlnQ0NRRHVVWjZuMEZWeU16QU5CZ2txaGtpRzl3MEJBUXNGQURBU01SQXdEZ1lEVlFRRERBZDAKWlhOMExXTmhNQjRYRFRFNE1EVXdOakl4TURRd09Wb1hEVEU0TURjd05USXhNRFF3T1Zvd0VqRVFNQTRHQTFVRQpBd3dIZEdWemRDMWpZVENDQVNJd0RRWUpLb1pJaHZjTkFRRUJCUUFEZ2dFUEFEQ0NBUW9DZ2dFQkFNQmpBS3dQCndhRUhwQTdaRW1iWWczaTNYNlppVmtGZFJGckJlTmFYTHFPL2R0RUdmWktqYUF0Wm45R1VsckQxZUlUS3UzVHgKOWlGVlV4Mmo1Z0tyWmpwWitCUnFiZ1BNbk5hS1hocmRTdDRtUUN0VFFZdGRYMVFZS0pUbWF5NU45N3FoNTZtWQprMllKRkpOWVhHWlJabkdMUXJQNk04VHZramF0ZnZOdmJ0WmtkY2orYlY3aWhXanp2d2theHRUVjZlUGxuM2p5CnJUeXBBTDliYnlVcHlad3E2MWQvb0Q4VUtwZ2lZM1dOWmN1YnNvSjhxWlRsTnN6UjVadEFJV0tjSE5ZbE93d2oKaG41RE1tSFpwZ0ZGNW14TU52akxPRUc0S0ZRU3laYlV2QzlZRUhLZTUxbGVxa1lmQmtBZWpPY002TnlWQUh1dApuay9DMHpXcGdENkIwbkVDQXdFQUFUQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFHTCtaNkRzK2R4WTZsU2VBClZHSkMvdzE1bHJ2ZXdia1YxN3hvcmlyNEMxVURJSXB6YXdCdFJRSGdSWXVtblVqOGo4T0hFWUFDUEthR3BTVUsKRDVuVWdzV0pMUUV0TDA2eTh6M3A0MDBrSlZFZW9xZlVnYjQrK1JLRVJrWmowWXR3NEN0WHhwOVMzVkd4NmNOQQozZVlqRnRQd2hoYWVEQmdma1hXQWtISXFDcEsrN3RYem9pRGpXbi8walI2VDcrSGlaNEZjZ1AzYnd3K3NjUDIyCjlDQVZ1ZFg4TWpEQ1hTcll0Y0ZINllBanlCSTJjbDhoSkJqa2E3aERpVC9DaFlEZlFFVFZDM3crQjBDYjF1NWcKdE03Z2NGcUw4OVdhMnp5UzdNdXk5bEthUDBvTXl1Ty82Tm1wNjNsVnRHeEZKSFh4WTN6M0lycGxlbTNZQThpTwpmbmlYZXc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==

 /accordion 
 accordion id="option-b" label="Option B—Bring Your Own Certificate: Signed by Recognized CA" 
如果您使用的是由认可的证书颁发机构签署的证书，则无需执行此步骤部分。
 /accordion 

### 8. 配置 FQDN

`<FQDN>` RKE配置文件中有一个引用。将此引用替换为您在 [3. Configure DNS](#3-configure-dns) 选择的FQDN 。

1. 打开 `rancher-cluster.yml`。

2. 在 `kind: Ingress` 与 `name: cattle-ingress-http:`

   替换 `<FQDN>` 为 [3. Configure DNS](#3-configure-dns)选择的FQDN 。

   **步骤结果:** 替换值后，文件应类似于以下示例（base64编码的字符串应不同）：

   ```
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
   ```

3) 保存文件并关闭它。

### 9. 配置Rancher版本

最后一个需要替换的参考是 `<RANCHER_VERSION>`。这需要替换为标记为稳定的Rancher版本。最新的Rancher稳定版本可以在 [GitHub README](https://github.com/rancher/rancher/blob/master/README.md)中找到。确保版本是实际的版本号，而不是诸如 `stable` 或 `latest`的命名标签。以下示例显示了配置为的版本 `v2.0.6`。

```
      spec:
        serviceAccountName: cattle-admin
        containers:
        - image: rancher/rancher:v2.0.6
          imagePullPolicy: Always
```

### 10. 备份您的RKE配置文件

关闭RKE配置文件 `rancher-cluster.yml`后, 将其备份到安全位置。升级Rancher时，可以再次使用此文件。

### 11. 运行RKE

完成所有配置后，使用RKE启动Rancher。您可以通过运行 `rke up` 命令并使用 `--config` 参数指向配置文件来完成此操作。

1. 在您的工作站上，确保 `rancher-cluster.yml` 与下载的 `rke` 二进制文件位于同一目录中。

2. 打开一个终端实例。转到包含配置文件和的目录 `rke`。

3. 输入下面的 `rke up` 命令之一。

   ```
   rke up --config rancher-cluster.yml
   ```

   **步骤结果:** 输出应与以下代码段相似：

   ```
   INFO[0000] Building Kubernetes cluster
   INFO[0000] [dialer] Setup tunnel for host [1.1.1.1]
   INFO[0000] [network] Deploying port listener containers
   INFO[0000] [network] Pulling image [alpine:latest] on host [1.1.1.1]
   ...
   INFO[0101] Finished building Kubernetes cluster successfully
   ```

### 12. 备份自动生成的配置文件

在安装过程中，RKE自动生成一个 `kube_config_rancher-cluster.yml` 与该 `rancher-cluster.yml` 文件位于同一目录中的配置文件。复制此文件并将其备份到安全位置。稍后在升级Rancher Server时将使用此文件。

### 下一步是什么？

- **推荐:** 查看 [创建备份-高可用性备份和还原](/docs/backups/backups/ha-backups/)，以了解在灾难情况下如何备份Rancher Server。
- 创建Kubernetes集群： [创建集群](/docs/tasks/clusters/creating-a-cluster/)。

<br/>

### 常见问题解答和故障排除

{{< ssl_faq_ha >}}
