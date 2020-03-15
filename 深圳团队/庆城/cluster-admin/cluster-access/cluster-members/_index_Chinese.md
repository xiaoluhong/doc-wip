---
标题: 向集群添加用户
---

如果希望为用户提供对集群中的 _所有_ 项目、节点和资源的访问和权限，请为该用户分配集群成员资格。

> **提示:** 希望为用户提供对集群中 _特定_ 项目的访问权限? 请参见 [添加项目成员](/docs/k8s-in-rancher/projects-and-namespaces/project-members/)。

以下两种情况下，您可以添加集群成员:

- 向新集群添加成员

  您可以在创建集群时将成员添加到集群中(如果可能，建议这样做)。

- [向现有集群添加成员](#editing-cluster-membership)

  在集群创建之后，随时可以向集群添加成员。

### 编辑集群成员

集群管理员可以编辑集群的成员关系，控制哪些Rancher用户可以访问集群以及他们可以使用哪些功能。

1.  在 **全局** 视图中，打开要向其添加成员的集群。

2.  从主菜单中选择 **成员**，然后单击 **添加成员**.

3.  搜索要添加到集群中的用户或组。

    如果配置了外部认证:

    - Rancher在您键入时从 [外部身份验证](/docs/admin-settings/authentication/) 源返回用户。

          	>**使用 AD 却找不到你的用户?**
          	>您的搜索属性配置可能存在问题.请参见 [配置Active Directory身份验证:步骤5](/docs/admin-settings/authentication/ad/).

    - 下拉菜单允许您添加组，而不是单个用户。下拉列表只列出登录用户所属的组。

          	>**注意:** 如果您是以本地用户登录的，那么外部用户不会显示在您的搜索结果中。更多信息，请参见 [外部身份验证配置和主要用户](/docs/admin-settings/authentication/#external-authentication-configuration-and-principal-users).

4.  给用户或组分配 **集群** 角色。

    [什么是集群角色?](/docs/admin-settings/rbac/cluster-project-roles/)

    > **提示：** 对于自定义角色，您可以修改可用于分配的各个角色的列表。
    >
    > - 将角色添加到列表中， [添加自定义角色](/docs/admin-settings/rbac/default-custom-roles/).
    > - 要从列表中删除角色， [锁定/解锁角色](/docs/admin-settings/rbac/locked-roles).

**结果：** 选择的用户被添加到集群中。

- 要撤消集群成员资格，请选择用户并单击 **删除**. 此操作删除的是成员关系，而不是用户。
- 要修改用户在集群中的角色，请从集群中删除它们，然后使用修改后的角色重新添加它们。

