---
标题: 配置活动目录(AD)
---

如果您的组织使用Microsoft活动目录作为中心用户存储库，则可以将Rancher配置为与活动目录服务器通信以对用户进行身份验证。这允许Rancher管理员基于在Active Directory中外部管理的用户和组控制对集群和项目的访问，同时允许最终用户在登录Rancher UI时使用其广告凭据进行身份验证。

Rancher使用LDAP与Active Directory服务器通信。因此，Active Directory的身份验证流与[OpenLDAP authentication](/docs/admin settings/authentication/OpenLDAP)集成的身份验证流相同。

> **注:**
>
> 在开始之前，请熟悉[外部身份验证配置和主体用户](/docs/admin-settings/authentication/#外部身份验证配置和主体用户)的概念。

### 先决条件

您需要创建或从您的广告管理员处获取一个新的广告用户，以用作Rancher的服务帐户。此用户必须具有足够的权限，才能执行LDAP搜索并读取您的AD域下的用户和组的属性。

通常(非管理员)**域用户**帐户应用于此目的，因为默认情况下，该用户对域分区中的大多数对象具有只读权限。

但是，请注意，在某些锁定的活动目录配置中，此默认行为可能不适用。在这种情况下，您需要确保服务帐户用户在基本OU(包含用户和组)上或全局范围内至少拥有**读取**和**列表内容**权限。

> **使用TLS？**
>
> 如果AD服务器使用的证书是自签名的或不是来自认可的证书颁发机构，请确保手头有PEM格式的CA证书(与任何中间证书连接)。您必须在配置期间粘贴此证书，以便Rancher能够验证证书链。

### 配置步骤
#### 打开活动目录配置

1. 使用初始本地`admin`帐户登录到Rancher UI。
2. 在**全局**视图中，导航到**安全** > **身份验证**
3. 选择**活动目录**。将显示**配置AD服务器**表单。

#### 配置活动目录服务器设置

在标题为`1`的部分中。配置一个Active Directory服务器，用特定于您的activedirectory服务器的信息填写字段。有关每个参数所需值的详细信息，请参阅下表。

> **注:**
>
> 如果您不确定要在“用户/组搜索库”字段中输入的正确值，请参阅[使用ldapsearch标识搜索库和架构](#附件使用ldapsearch标识搜索库和架构)。

**表1:AD服务器参数**

| 参数 | 说明 |
|:--|:--|
| Hostname  | 指定AD服务器的主机名或IP地址 |
| 端口 | 指定活动目录服务器侦听连接的端口。未加密的LDAP通常使用389的标准端口，而LDAPS使用636端口 |
| TLS  | 选中此框可启用SSL/TLS上的LDAP(通常称为LDAPS) |
| 服务器连接超时 |  Rancher在认为无法访问AD服务器之前等待的持续时间(秒)。 |
| 服务帐户用户名 | 输入对域分区具有只读访问权限的AD帐户的用户名(请参阅[先决条件](#先决条件))。用户名可以用NetBIOS格式(例如“域\服务帐户”)或UPN格式(例如“服务帐户@DOMAIN.com”)输入。 |
| 服务帐户密码 | 服务帐户的密码。 |
| 默认登录域 | 当您使用AD域的NetBIOS名称配置此字段时，当绑定到AD服务器时，在没有域的情况下输入的用户名(例如“jdoe”)将自动转换为斜杠的NetBIOS登录(例如“Login | Domain\jdoe”)。如果您的用户以UPN(例如“jdoe@acme.com”)作为用户名进行身份验证，则此字段**必须保留为空。 |
| 用户搜索库 | 目录树中开始搜索用户对象的节点的可分辨名称。所有用户都必须是此基本DN的后代。例如:“ou=people，dc=acme，dc=com” |
| 组搜索库 | 如果组位于`用户搜索库`下配置的节点之外的其他节点下，则需要在此处提供可分辨名称。否则就空着。例如:`ou=groups，dc=acme，dc=com` |

---

#### 配置用户/组架构

在标题为`2`的部分中。自定义架构必须为Rancher提供与目录中使用的架构对应的用户和组属性的正确映射。

Rancher使用LDAP查询来搜索和检索关于Active Directory中的用户和组的信息。本节中配置的属性映射用于构造搜索筛选器和解析组成员身份。因此，最重要的是所提供的设置反映您的广告领域的现实。
> **注:**
>
> 如果您不熟悉活动目录域中使用的架构，请参阅[使用ldapsearch标识搜索库和架构](#附件使用ldapsearch标识搜索库和架构)以确定正确的配置值。

##### 用户架构

下表详细说明了用户架构部分配置的参数。

**表2:用户架构配置参数**

| 参数 | 说明 |
|:--|:--|
| 对象类 | 用于域中用户对象的对象类的名称。如果已定义，请仅指定对象类的名称-*不要*将其包含在LDAP包装中，如&(objectClass=xxxx) |
| 用户名属性 | 其值适合作为显示名称的用户属性。 |
| 登录属性 | 其值与用户登录Rancher时输入的凭据的用户名部分匹配的属性。如果您的用户以其UPN(例如`jdoe@acme.com`)作为用户名进行身份验证，则此字段通常必须设置为`userPrincipalName`。否则，对于旧的NetBIOS风格的登录名(例如`jdoe`)，通常是`sAMAccountName`。 |
| 用户成员属性 | 包含用户所属组的属性。 |
| 搜索属性 | 当用户输入文本以在用户界面中添加用户或组时，Rancher会查询广告服务器，并尝试根据此设置中提供的属性匹配用户。可以通过使用管道(`\ | `)符号分隔属性来指定多个属性。要匹配UPN用户名(例如jdoe@acme.com)，通常应将此字段的值设置为`userPrincipalName`。 |
| 搜索筛选器 | 当Rancher尝试将用户添加到网站访问列表或尝试将成员添加到群集或项目时，此筛选器将应用于搜索的用户列表。例如，用户搜索过滤器可以是<code>(&#124；(memberOf=CN=group1，CN=Users，DC=testad，DC=rancher，DC=io)(memberOf=CN=group2，CN=Users，DC=testad，DC=rancher，DC=io))</code>。注意:如果搜索筛选器未使用[有效的广告搜索语法]，则用户列表将为空。 |
| 用户启用属性 | 包含表示用户帐户标志按位枚举的整数值的属性。Rancher使用此选项来确定用户帐户是否已禁用。通常应将此设置保留为广告标准`userAccountControl`。 |
| 禁用状态位掩码 | 这是指定禁用用户帐户的`用户启用属性`的值。通常，您应该将此设置保留为Microsoft活动目录架构中指定的默认值`2`(请参见[此处](https://docs.Microsoft.com/en-us/windows/desktop/adschema/a-useraccountcontrol 35; remarks))。 |

---

##### 组架构

下表详细说明了组架构配置的参数。

**表3:组架构配置参数**

| 参数 | 说明 |
|:--|:--|
| 对象类 | 用于在域中对对象进行分组的对象类的名称。如果已定义，请仅指定对象类的名称-*不要*将其包含在LDAP包装中，如&(objectClass=xxxx) |
| 名称属性 | 其值适合显示名称的组属性。 |
| 组成员用户属性 | **用户属性**的名称，其格式与`组成员映射属性`中的组成员匹配。 |
| 组成员映射属性 | 包含组成员的组属性的名称。 |
| 搜索属性 | 用于在将组添加到群集或项目时构造搜索筛选器的属性。请参见用户架构`Search Attribute`的说明。 |
| 搜索筛选器 | 当Rancher尝试将组添加到站点访问列表或尝试将组添加到群集或项目时，此筛选器将应用于搜索的组列表。例如，组搜索筛选器可以是<code>(&#124；(cn=group1)(cn=group2))</code>。注意:如果搜索筛选器未使用[有效的AD搜索语法](https://docs.microsoft.com/en-us/windows/win32/adsi/search-filter-syntax)，则组列表将为空。 |
| 组DN属性 | 其格式与描述用户成员身份的用户属性中的值匹配的组属性的名称。请参见`用户成员属性`。 |
| 嵌套组成员身份 | 此设置定义Rancher是否应解析嵌套组成员身份。仅当您的组织使用这些嵌套成员身份时才使用(即您有包含其他组作为成员的组)。 |

---

#### 测试身份验证

完成配置后，请测试与AD服务器的连接。如果测试成功，将隐式启用与配置的活动目录的身份验证。

> **注:**
>
> 与此步骤中输入的凭据相关的AD用户将映射到本地主体帐户并在Rancher中分配管理员权限。因此，您应该有意识地决定使用哪个广告帐户来执行此步骤。

1. 输入应映射到本地主体帐户的广告帐户的**用户名**和**密码**。
2. 单击**验证活动目录**以完成设置。

**结果:**

- 已启用活动目录身份验证。
- 您已使用提供的广告凭据以管理员身份登录到Rancher。

> **注:**
>
> 如果LDAP服务中断，您仍然可以使用本地配置的`admin`帐户和密码登录。

### 附件:使用ldapsearch标识搜索库和模式

为了成功配置AD身份验证，必须提供与AD服务器的层次结构和架构相关的正确配置。

[`ldapsearch`](http://manpages.ubuntu.com/manpages/artful/man1/ldapsearch.1.html)工具允许您查询您的广告服务器以了解用于用户和组对象的架构。

对于下面提供的示例命令，我们假设:

- 活动目录服务器的主机名为`ad.acme.com`
- 服务器正在监听端口`389上的未加密连接`
- 活动目录域是`acme`
- 您有一个用户名为`jdoe`和密码为`secret`的有效广告帐户`

#### 识别搜索库

首先，我们将使用`ldapsearch`来标识用户和组的父节点的可分辨名称(DN):

```
$ ldapsearch -x -D "acme\jdoe" -w "secret" -p 389 \
-h ad.acme.com -b "dc=acme,dc=com" -s sub "sAMAccountName=jdoe"
```

此命令执行LDAP搜索，搜索基设置为域根目录(`-b dc=acme，dc=com`)，并执行针对用户帐户的筛选器(`sAMAccountNam=jdoe`)，返回所述用户的属性:

![LDAP User](/img/rancher/ldapsearch-user.png)

因为在这种情况下，用户的DN是`CN=John Doe，CN=user s，DC=acme，DC=com`[5]，所以我们应该使用父节点DN`CN=Users，DC=acme，DC=com`配置**用户搜索库**。

同样，基于**memberOf**属性[4]中引用的组的DN，**组搜索基**的正确值将是该值的父节点，即`OU=Groups，DC=acme，DC=com`。

#### 标识用户架构

上述`ldapsearch`查询的输出还允许确定在用户架构配置中使用的正确值:

- `对象类`:**个人**[1]
- `用户名属性`:**名称**[2]
- `登录属性`:**sAMAccountName**[3]
- `用户成员属性`:**成员**[4]

> **注:**
>
> 如果我们组织中的广告用户使用其UPN(例如jdoe@acme.com)而不是短登录名进行身份验证，则我们必须将`Login Attribute`设置为**userPrincipalName**。

我们还将`Search Attribute`参数设置为**sAMAccountName  |  name**。这样，用户可以通过输入用户名或全名添加到Rancher UI中的集群/项目中。

#### 标识组架构

接下来，我们将查询与此用户关联的组之一，在本例中为`CN=examplegroup，OU=groups，DC=acme，DC=com`:

```
$ ldapsearch -x -D "acme\jdoe" -w "secret" -p 389 \
-h ad.acme.com -b "ou=groups,dc=acme,dc=com" \
-s sub "CN=examplegroup"
```

此命令将通知我们用于组对象的属性:

![LDAP Group](/img/rancher/ldapsearch-group.png)

同样，这允许我们确定要在组架构配置中输入的正确值:

- `对象类`: **组**[1]
- `名称属性`: **名称**[2]
- `组成员映射属性`: **成员**[3]
- `搜索属性`: **sAMAccountName**[4]

查看**member**属性的值，我们可以看到它包含被引用用户的DN。这对应于我们的用户对象中的**distinguishedName**属性。因此，必须将`Group Member User Attribute`参数的值设置为此属性。

同样，我们可以看到用户对象中**memberOf**属性中的值对应于组的**distinguishedName**[5]。因此，我们需要将`Group DN Attribute`参数的值设置为此属性。

### 附件:故障排除

如果在测试与Active Directory服务器的连接时遇到问题，请首先仔细检查为服务帐户输入的凭据以及搜索基础配置。您还可以检查Rancher日志，以帮助查明问题的原因。调试日志可能包含有关错误的更详细信息。请参阅本文档中的[如何启用调试日志](/docs/faq/technical/#How-can-I-enable-debug-logging)。

