---
标题: 访问集群
---

您可以通过多种方式与由Rancher管理的Kubernetes集群进行交互:

- **Rancher UI**

  Rancher为与集群交互提供了直观的用户界面。UI中所有可用的选项都调用Rancher API。因此，在UI中任何可行的操作在Rancher CLI或Rancher API中也是可行的。

- **kubectl**

  您可以使用Kubernetes命令行工具[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)来管理你的集群，使用kubectl有两种选择:

  - **Rancher kubectl shell**

    通过启动Rancher UI中可用的kubectl shell与集群进行交互。此选项不需要您进行任何配置操作。

    有关更多信息，请参见 [使用kubectl Shell访问集群](/docs/k8s-in-rancher/kubectl/#accessing-clusters-with-kubectl-shell).

  - **终端远程连接**

    您还可以通过在本地桌面安装 [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) 来与集群进行交互，然后将集群的kubeconfig文件复制到本地 `~/.kube/config` 目录中。

    有关更多信息，请参见 [使用kubectl和kubeconfig文件访问集群](/docs/k8s-in-rancher/kubectl/#accessing-clusters-with-kubectl-and-a-kubeconfig-file).

- **Rancher CLI**

  您可以通过下载Rancher自己的CLI来控制集群, [Rancher CLI](/docs/cli/). 这个CLI工具可以直接与不同的集群和项目交互，或者向它们传递 `kubectl` 命令。

- **Rancher API**

  最后，您可以通过Rancher API与集群进行交互。在使用API之前，您必须获得一个 [API key](/docs/user-settings/api-keys/). 要查看API对象的不同资源字段和操作，请打开API UI，可以通过单击 **View in API** 来访问任何Rancher UI对象
