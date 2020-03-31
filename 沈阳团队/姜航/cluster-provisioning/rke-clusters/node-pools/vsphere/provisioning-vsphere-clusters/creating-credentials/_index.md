---
title: 在vSphere控制台中创建凭证
---

本节介绍了如何创建vSphere用户名和密码。 您需要将这些vSphere凭证提供给Rancher，以便Rancher可以在vSphere中创建资源。

下面的表格列出了vSphere用户账户需要的权限：

| Privilege Group | Operations                                                                                                                                         |
| :-------------- | :------------------------------------------------------------------------------------------------------------------------------------------------- |
| Datastore       | AllocateSpace <br /> Browse <br /> FileManagement (Low level file operations) <br /> UpdateVirtualMachineFiles <br /> UpdateVirtualMachineMetadata |
| Network         | Assign                                                                                                                                             |
| Resource        | AssignVMToPool                                                                                                                                     |
| Virtual Machine | Config (All) <br /> GuestOperations (All) <br /> Interact (All) <br /> Inventory (All) <br /> Provisioning (All)                                   |

以下步骤将创建具有所需权限的角色，并在vSphere控制台中将其分配给指定用户：

1. 从**vSphere**控制台中, 进入**Administration**页面.

2. 进入**Roles**选项卡.

3. 创建一个新角色并命名，然后选择上面的表格中列出的权限。

   ![image](/img/rancher/rancherroles1.png")

4. 选择 **Users and Groups** 选项卡.

5. 创建一个新用户。填写表单，然后单击**OK**。 请确保记下用户名和密码，因为在Rancher中配置主机模板时将需要它。

   ![image](/img/rancher/rancheruser.png")

6. 选择 **Global Permissions** 选项卡.

7. 创建一个新的全局权限。添加您先前创建的用户，并将先前创建的角色应用到该用户。 点击**OK**。

   ![image](/img/rancher/globalpermissionuser.png")

   ![image](/img/rancher/globalpermissionrole.png")

**结果:** 现在，您具有可用于操作vSphere资源的凭证。
