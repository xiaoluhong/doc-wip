---
title: 使用防火墙打开端口
---

Linux的一些发行版[源自RHEL](https://en.wikipedia.org/wiki/Red_Hat_Enterprise_Linux#Rebuilds)，包括Oracle Linux，可能有默认的防火墙规则来阻止与Helm的通信。

例如，AWS中的一个Oracle Linux映像具有REJECT规则，可阻止Helm与Tiller通信：

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     icmp --  anywhere             anywhere
ACCEPT     all  --  anywhere             anywhere
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

您可以使用以下命令检查默认防火墙规则：

```
sudo iptables --list
```

本节介绍如何使用`firewalld`为高可用性Rancher服务器群集中的节点应用[防火墙端口规则](/docs/installation/references)。

## 先决条件

安装 v7.x 或更高版本的`firewalld`：

```
yum install firewalld
systemctl start firewalld
systemctl enable firewalld
```

## 应用防火墙端口规则

在Rancher高可用性安装说明中，Rancher服务器设置在三个节点上，这三个节点具有Kubernetes的所有三个角色：etcd、controlplane和worker。如果您的Rancher服务器节点具有所有这三个角色，请在每个节点上运行以下命令：

```
firewall-cmd --permanent --add-port=22/tcp
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --permanent --add-port=2376/tcp
firewall-cmd --permanent --add-port=2379/tcp
firewall-cmd --permanent --add-port=2380/tcp
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=8472/udp
firewall-cmd --permanent --add-port=9099/tcp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10254/tcp
firewall-cmd --permanent --add-port=30000-32767/tcp
firewall-cmd --permanent --add-port=30000-32767/udp
```

如果您的Rancher服务器节点具有单独的角色，请根据节点的角色使用以下命令：

```
## 对于etcd节点，运行以下命令：
firewall-cmd --permanent --add-port=2376/tcp
firewall-cmd --permanent --add-port=2379/tcp
firewall-cmd --permanent --add-port=2380/tcp
firewall-cmd --permanent --add-port=8472/udp
firewall-cmd --permanent --add-port=9099/tcp
firewall-cmd --permanent --add-port=10250/tcp

## 对于control plane节点，运行以下命令：
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --permanent --add-port=2376/tcp
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=8472/udp
firewall-cmd --permanent --add-port=9099/tcp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10254/tcp
firewall-cmd --permanent --add-port=30000-32767/tcp
firewall-cmd --permanent --add-port=30000-32767/udp

## 对于worker nodes节点，运行以下命令：
firewall-cmd --permanent --add-port=22/tcp
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --permanent --add-port=2376/tcp
firewall-cmd --permanent --add-port=8472/udp
firewall-cmd --permanent --add-port=9099/tcp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --permanent --add-port=10254/tcp
firewall-cmd --permanent --add-port=30000-32767/tcp
firewall-cmd --permanent --add-port=30000-32767/udp
```

在节点上运行`firewall-cmd`命令后，使用以下命令启用防火墙规则：


```
firewall-cmd --reload
```

**结果：** 防火墙已更新，因此Helm可以与Rancher服务器节点通信。

