---
标题: RKE模板和基础设施
---

在Rancher中，RKE模板用于提供Kubernetes和定义Rancher设置，而节点模板用于提供节点。

因此，即使启用了RKE模板强制，最终用户在创建Rancher集群时仍然可以灵活地选择底层硬件。RKE模板的最终用户仍然可以选择基础设施提供商和他们想要使用的节点。

如果要标准化集群中的硬件，请将RKE模板与节点模板或服务器配置工具(如Terraform)结合使用。

#### 节点模板

[节点模板](/docs/user-settings/node-templates)负责Rancher中的节点配置和节点设置。从用户配置文件中，可以设置节点模板以定义在每个节点池中使用的模板。启用节点池后，可以确保每个节点池中都有所需数量的节点，并确保池中的所有节点都相同。

#### Terraform

Terraform是一个服务器配置工具。它使用基础设施作为代码，允许您使用Terraform配置文件创建基础设施的几乎每个方面。它可以自动执行服务器配置过程，这种方式是自文档化的，并且在版本控制中易于跟踪。

本节重点介绍如何将Terraform与[Rancher 2 Terraform provider](https://www.terraform.io/docs/providers/rancher2/)一起使用，这是标准化Kubernetes集群硬件的推荐选项。如果使用Rancher Terraform提供程序提供硬件，然后使用RKE模板在该硬件上提供Kubernetes集群，则可以快速创建一个全面的、可用于生产的集群。

Terraform允许您:

- 将几乎任何类型的基础设施定义为代码，包括服务器、数据库、负载平衡器、监视、防火墙设置和SSL证书
- 利用目录应用程序和多群集应用程序
- 跨多个平台(包括Rancher和主要云提供商)对基础设施进行编码
- 将基础结构作为代码提交到版本控制
- 轻松重复配置和设置基础结构
- 将基础设施更改纳入标准开发实践
- 防止配置漂移，其中一些服务器的配置与其他服务器不同

## Terraform是如何工作的？

Terraform是用扩展名为`.tf`的文件编写的。它是用HashiCorp配置语言编写的，HashiCorp配置语言是一种声明性语言，允许您在集群中定义所需的基础设施、正在使用的云提供程序以及提供程序的凭据。然后Terraform向提供者发出API调用，以便有效地创建基础设施。

要使用Terraform创建Rancher配置的集群，请转到Terraform配置文件并将提供程序定义为Rancher 2。您可以使用Rancher API密钥设置Rancher 2提供程序。注意:API密钥具有与其关联的用户相同的权限和访问级别。

然后Terraform调用Rancher API来提供基础设施，Rancher调用基础设施提供者。例如，如果您想使用Rancher在AWS上提供基础设施，您可以在Terraform配置文件或环境变量中提供Rancher API密钥和AWS凭据，以便它们可以用于提供基础设施。

当需要更改基础结构时，您可以在Terraform配置文件中进行更改，而不是手动更新服务器。然后，可以将这些文件提交到版本控制、验证，并根据需要进行审阅。然后当您运行`terraform apply`时，将部署更改。

## 使用Terraform的技巧

- 在[documentation for the Rancher 2 provider.](https://www.terraform.io/docs/providers/rancher2/)

- 在Terraform设置中，可以使用Docker Machine节点驱动程序安装Docker Machine。

- 也可以在Terraform提供程序中修改auth。

- 通过更改Rancher中的设置，然后返回并检查Terraform状态文件，查看它如何映射到基础结构的当前状态，可以反向工程如何在Terraform中定义设置。

- 如果您想在一个地方管理Kubernetes集群设置、Rancher设置和硬件设置，请使用[Terraform modules](https://github.com/rancher/terraform-modules)。您可以将集群配置YAML文件或RKE模板配置文件传递给Terraform模块，以便Terraform模块创建它。在这种情况下，可以将基础结构用作代码来管理Kubernetes集群及其底层硬件的版本控制和修订历史。

## 创建符合CIS基准的集群的技巧

本节描述了一种方法，可以使安全性和符合性相关的配置文件成为集群中的标准配置文件。

创建[CIS基准兼容群集时，](/docs/security/)有一个加密配置文件和一个审核日志配置文件。
您的基础架构配置系统可以将这些文件写入磁盘。然后在RKE模板中，指定这些文件的位置，然后将加密配置文件和审计日志配置文件作为额外的挂载添加到`kube api服务器`。

然后确保RKE模板中的`kube api server`标志使用符合CIS的配置文件。

通过这种方式，可以创建符合CIS基准的标志。

## 资源

- [Terraform文档](https://www.terraform.io/docs/)
- [Rancher2 Terraform提供程序文档](https://www.terraform.io/docs/providers/rancher2/)
- [RanchCast- 第1集:Rancher 2 Terraform Provider](https://youtu.be/YNCq-prI8-8):在本演示中，社区主管Jason van Brackel使用Rancher 2 Terraform Provider提供节点并创建自定义集群。