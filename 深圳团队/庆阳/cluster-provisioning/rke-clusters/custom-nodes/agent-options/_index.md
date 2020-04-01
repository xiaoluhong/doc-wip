---
标题: Rancher Agent选项
---

Rancher在每个节点上部署一个agent来与节点通信。 本页描述了可以传递给agent的选项。 要使用这些选项，您需要[使用自定义节点创建集群](/docs/cluster-provisioning/rke-clusters/custom-nodes/)并在添加节点时将选项添加到生成的`docker run`命令中。

有关Rancher如何使用节点agent与下游集群通信的概述，请参阅[体系结构部分。](/docs/overview/architecture/#3-node-agent)

### 通用选项

| 参数       | 环境变量 | 描述                                                                                                         |
| --------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `--server`      | `CATTLE_SERVER`      | agent连接到已配置的Rancher `server-url`                                         |
| `--token`       | `CATTLE_TOKEN`       | 在Rancher中注册节点所需的token                                                               |
| `--ca-checksum` | `CATTLE_CA_CHECKSUM` | 使用已配置的Rancher`cacerts`进行SHA256校验和来验证                                         |
| `--node-name`   | `CATTLE_NODE_NAME`   | 重写注册节点的主机名（默认为`hostname -s`)                                 |
| `--label`       | `CATTLE_NODE_LABEL`  | 向节点添加节点标签。 对于多个标签，请传递额外的`--label`选项。 (`--label key=value`)          |
| `--taints`      | `CATTLE_NODE_TAINTS` | 将节点taints添加到节点。 对于多个taints，请传递额外的`--taints`选项。 ('--taints key=value:effect`)|

### 角色选项

| 参数        | 环境变量 | 描述                                                  |
| ---------------- | -------------------- | ------------------------------------------------------------ |
| `--all-roles`    | `ALL=true`           | 将所有角色(`etcd`,`controlplane`,`worker`)应用到节点 |
| `--etcd`         | `ETCD=true`          | 将角色`etcd`应用到节点                            |
| `--controlplane` | `CONTROL=true`       | 将角色`controlplane`应用到节点                     |
| `--worker`       | `WORKER=true`        | 将角色`worker`应用到节点                           |

### IP地址选项

| 参数            | 环境变量      | 描述                                                                                  |
| -------------------- | ------------------------- | -------------------------------------------------------------------------------------------- |
| `--address`          | `CATTLE_ADDRESS`          | 该节点将注册的IP地址（默认为`8.8.8.8`的IP)|
| `--internal-address` | `CATTLE_INTERNAL_ADDRESS` | 专用网络上用于主机间通信的IP地址                        |

#### 动态IP地址选项

出于自动化的目的，您不能在命令中具有特定的IP地址，因为它必须是通用的才能用于每个节点。 为此，我们有动态IP地址选项。 它们用作现有IP地址选项的值。 支持`--address`和`--internal-address`。

| 值          | 例子                 | 描述                                                                                                                                                           |
| -------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 接口名称 | `--address eth0`        | 第一个配置的IP地址将从给定的接口中检索                                                                                            |
| `ipify`        | `--address ipify`       | 从`https://api.ipify.org`获取的值将被使用                                                                                                             |
| `awslocal`     | `--address awslocal`    | 从`http://169.254.169.254/latest/meta-data/local-ipv4`获取的值将被使用                                                                                |
| `awspublic`    | `--address awspublic`   | 从`http://169.254.169.254/latest/meta-data/public-ipv4`获取的值将被使用                                                                               |
| `doprivate`    | `--address doprivate`   | 从`http://169.254.169.254/metadata/v1/interfaces/private/0/ipv4/address`获取的值将被使用                                                              |
| `dopublic`     | `--address dopublic`    | 从`http://169.254.169.254/metadata/v1/interfaces/public/0/ipv4/address`获取的值将被使用                                                               |
| `azprivate`    | `--address azprivate`   | 从`http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/privateIpAddress?api-version=2017-08-01&format=text`获取的值将被使用 |
| `azpublic`     | `--address azpublic`    | 从`http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-08-01&format=text`获取的值将被使用  |
| `gceinternal`  | `--address gceinternal` | 从`http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip`获取的值将被使用                                               |
| `gceexternal`  | `--address gceexternal` | 从`http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/access-configs/0/external-ip`获取的值将被使用                     |
| `packetlocal`  | `--address packetlocal` | 从`https://metadata.packet.net/2009-04-04/meta-data/local-ipv4`获取的值将被使用                                                                       |
| `packetpublic` | `--address packetlocal` | 从`https://metadata.packet.net/2009-04-04/meta-data/public-ipv4`获取的值将被使用                                                                      |
