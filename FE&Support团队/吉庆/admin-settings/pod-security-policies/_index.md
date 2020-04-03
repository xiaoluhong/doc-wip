---
标题：Pod安全策略
---

_Pod安全策略_(或PSP)是控制Pod规范对安全性敏感的方面(如root特权)的对象。如果Pod不符合PSP中指定的条件，Kubernetes将不允许其启动，并且Rancher将显示错误消息`Pod <NAME>被禁止：无法验证...`。

- 您可以在集群或项目级别分配PSP。
- PSP通过继承工作。

    - 默认情况下，分配给集群的PSP由其项目以及添加到这些项目的任何名称空间继承。
    - **例外：** 未分配给项目的命名空间不会继承PSP，无论PSP是分配给集群还是项目。由于这些名称空间没有PSP，因此将工作负载部署到这些名称空间将失败，这是Kubernetes的默认行为。
    - 您可以通过直接向项目分配其他PSP来覆盖默认PSP。

    - 在分配PSP之前，群集或项目中已经在运行的任何工作负荷是否符合PSP的规定，将不会进行检查。需要克隆或升级工作负载以查看它们是否通过了PSP。

> **注意：** 必须先在集群级别启用PSP，然后才能将它们分配给项目。可以通过[编辑集群。](/docs/cluster-admin/editing-clusters/)进行配置。

在[Kubernetes文档](https://kubernetes.io/docs/concepts/policy/pod-security-policy/)中了解有关Pod安全策略的更多信息。

> **最佳实践：** 在集群级别设置Pod安全性。

使用Rancher，您可以使用我们的GUI创建Pod安全策略，而不是创建YAML文件。

### 默认Pod安全策略

_自v2.0.7起可用_

Rancher随附了两个默认的Pod安全策略(PSP)：`受限`和`不受限制`策略。

- `受限制的`

```
该策略基于Kubernetes [示例受限策略](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/policy/restricted-psp.yaml)。它极大地限制了可以将哪些类型的Pod部署到集群或项目。这项政策：

- 阻止Pod以特权用户身份运行，并防止特权升级。
- 验证服务器所需的安全性机制是否到位(例如，限制只能将哪些卷安装到核心卷类型，以及防止添加根补充组)。
```

- `无限制`

```
该策略等效于在禁用PSP控制器的情况下运行Kubernetes。对于可以将哪些Pod部署到集群或项目中，它没有任何限制。
```

### 创建Pod安全策略

1. 在`全局`视图中，从主菜单中选择`安全性`>` Pod安全策略`。然后点击`添加策略`。

**步骤结果：** 将打开`添加策略`表单。

2. 命名策略。

3. 填写表格的每个部分。请参阅下面链接的Kubernetes文档，以获取有关每个策略的作用的更多信息。

- 基本政策：

```
- [特权升级](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#privilege-escalation)
- [主机命名空间] [2]
- [只读根文件系统] [1]
```

- [Capability Policies](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#capabilities)
- [Volume Policy][1]
- [Allowed Host Paths Policy][1]
- [FS Group Policy][1]
- [Host Ports Policy][2]
- [Run As User Policy][3]
- [SELinux Policy](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#selinux)
- [Supplemental Groups Policy][3]

#### 下一步是什么？

您可以在以下上下文中添加Pod安全策略(以下简称PSP)：

- [创建集群时](/docs/cluster-provisioning/rke-clusters/options/pod-security-policies/)
- [编辑现有群集时](/docs/k8s-in-rancher/editing-clusters/)
- [创建项目时](/docs/k8s-in-rancher/项目和命名空间/＃creating-a-project/)
- [编辑现有项目时](/docs/k8s-in-rancher/projects-and-namespaces/editing-projects/)

> **注意：** 我们建议在群集和项目创建期间添加PSP，而不是将其添加到现有的PSP中。
