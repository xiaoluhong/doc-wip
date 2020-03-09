---
title: 常见问题
---

#### canal Pods 显示 READY 2/3

此问题的最常见原因是节点之间的8472/UDP端口没有打开。检查您的本地防火墙，网络路由和安全组。

一旦解决网络问题后，`canal` pods应超时并重新启动以建立其连接。

#### nginx-ingress-controller Pods 显示 RESTARTS

造成此问题的最常见原因是 `canal` pods未能建立overlay网络。 参阅 [canal Pods 显示 READY `2/3`](#canal-pods-show-ready-2-3) 来处理。

#### Failed to set up SSH tunneling for host [xxx.xxx.xxx.xxx]: Can't retrieve Docker Info

##### Failed to dial to /var/run/docker.sock: ssh: rejected: administratively prohibited (open failed)

- 指定连接的用户无权访问Docker套接字。可以通过主机日志或者运行命令 `docker ps` 来检查:

```
$ ssh user@server
user@server$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
```

如何正确设置，参阅 [使用非root用户管理Docker](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)。

- 当使用RedHat/CentOS作为操作系统时，由于[Bugzilla＃1527565](https://bugzilla.redhat.com/show_bug.cgi?id=1527565)，您不能使用`root`用户来连接节点。 您将需要为访问Docker套接字单独配置一个用户。 参阅 [使用非root用户管理Docker](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)。

- SSH服务器版本不是6.7或更高版本。这是套接字转发正常工作所必需的，套接字转发用于通过SSH连接到Docker套接字。可以使用要连接的主机上的`sshd -V`或使用netcat来检查:

```
$ nc xxx.xxx.xxx.xxx 22
SSH-2.0-OpenSSH_6.6.1p1 Ubuntu-2ubuntu2.10
```

##### Failed to dial ssh using address [xxx.xxx.xxx.xxx:xx]: Error configuring SSH: ssh: no key found

- `ssh_key_path` 文件中指定的key文件无法被访问。确保指定了私钥文件（而不是公共密钥`.pub`），并且正在运行`rke`命令的用户可以访问私钥文件。

##### Failed to dial ssh using address [xxx.xxx.xxx.xxx:xx]: ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey], no supported methods remain

- 对于正在访问的节点，文件中指定的密钥文件 `ssh_key_path` 不正确。 仔细检查是否为节点指定了正确的`ssh_key_path`，以及是否指定了与之连接的正确用户。

##### Failed to dial ssh using address [xxx.xxx.xxx.xxx:xx]: Error configuring SSH: ssh: cannot decode encrypted private keys

- 如果要使用加密的私钥，则应使用`ssh-agent`来用密码加载密钥。如果在运行`rke`命令的环境中找到`SSH_AUTH_SOCK`环境变量，它将自动用于连接到节点。

##### Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?

- 在已配置的`address`和`port`上无法访问该节点。
