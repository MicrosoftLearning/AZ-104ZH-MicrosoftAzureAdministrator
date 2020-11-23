---
lab:
    title: '02a - 管理订阅和 RBAC'
    module: '模块 02 - 治理与符合性'
---

# 实验室 02a - 管理订阅和 RBAC
# 学生实验室手册

## 实验室场景

为了改善 Contoso 中的 Azure 资源管理，你被要求实现以下功能：

- 创建一个将包括所有 Contoso Azure 订阅的管理组

- 授予将管理组中所有订阅的支持申请提交给指定的 Azure Active Directory 用户的权限。该用户的权限应仅限于： 

    - 创建支持请求工单
    - 查看资源组 

## 目标

在本实验室中，你将：

+ 任务 1：实现管理组
+ 任务 2：创建自定义 RBAC 角色 
+ 任务 3：分配 RBAC 角色


## 预计用时：30 分钟

## 说明

### 练习 1：

#### 任务 1：实现管理组

在此任务中，你将需要创建和配置管理组。 

1. 登录到 [Azure 门户](https://portal.azure.com)。

1. 搜索并选择 **“管理组”**，然后在 **“管理组”** 边栏选项卡上，单击 **“+ 添加管理组”**。

    >**注意**：如果你以前尚未创建管理组，请选择 **“开始使用管理组”**

1. 使用以下设置创建一个新的管理组：

    | 设置 | 数值 |
    | --- | --- |
    | 管理组 ID | **az104-02-mg1**|
    | 管理组显示名称 | **az104-02-mg1**|

1. 在管理组列表中，单击表示新建管理组的条目，然后显示其 **“详细信息”**。

1. 从 **"az104-02-mg1"** 边栏选项卡，单击 **“+添加订阅”** 并将你在本实验室中使用的订阅添加到管理组。

    >**说明**：将 Azure 订阅的 ID 复制到剪贴板。你将在下一个任务中用到它。

#### 任务 2：创建自定义 RBAC 角色

在此任务中，你创建自定义 RBAC 角色的定义。

1. 在实验室计算机上，在记事本中打开名为 **“\Allfiles\Labs\02\az104-02a-customRoleDefinition.json”** 的文件查看其内容：

   ```json
   {
      "Name": "Support Request Contributor (Custom)",
      "IsCustom": true,
      "Description": "Allows to create support requests",
      "Actions": [
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Support/*"
      ],
      "NotActions": [
      ],
      "AssignableScopes": [
          "/providers/Microsoft.Management/managementGroups/az104-02-mg1",
          "/subscriptions/SUBSCRIPTION_ID"
      ]
   }
   ```

1. 用复制到剪贴板中的订阅 ID 替换 JSON 文件中的 `SUBSCRIPTION_ID` 占位符，然后保存更改。

1. 在 Azure 门户中，直接单击搜索文本框右侧的工具栏图标，打开 **“Cloud Shell”** 窗格。

1. 如果提示需要选择 **“Bash”** 或 **“PowerShell”**，请选择 **“PowerShell”**。 

    >**说明**：如果这是你第一次启动 **“Cloud Shell”**，并看到 **“你没有安装存储”** 的消息，请选择你在本实验室中使用的订阅，然后单击 **“创建存储”**。 

1. 在“Cloud Shell”窗格的工具栏中，单击 **“上传/下载文件”** 图标，在下拉菜单中，单击 **“上传”**，然后上传文件 **\\Allfiles\\Labs\\02\\az104-02a-customRoleDefinition.json”** 到 Cloud Shell 主目录中。

1. 在“Cloud Shell”窗格中，运行下列命令创建自定义角色定义：

   ```pwsh
   New-AzRoleDefinition -InputFile $HOME/az104-02a-customRoleDefinition.json
   ```

1. 关闭“Cloud Shell”窗格。

#### 任务 3：分配 RBAC 角色

在此任务中，你将创建一个 Azure Active Dicrectory 用户，将在上一个任务中创建的 RBAC 角色分配给该用户，并验证该用户可以执行 RBAC 角色定义中指定的任务。

1. 在 Azure 门户中，搜索并选择 **“Azure Active Directory”**，在“Azure Active Directory”边栏选项卡上单击 **“用户”**，然后单击 **“+ 新建用户”**。

1. 使用以下设置创建一个新用户（保留其他设置为默认值）：

    | 设置 | 数值 |
    | --- | --- |
    | 用户名称 | **az104-02-aaduser1**|
    | 名称 | **az104-02-aaduser1**|
    | 我来创建密码 | 已启用 |
    | 初始密码 | **Pa55w.rd124** |

    >**说明**：将完整的 **用户名复制到剪贴板**。稍后会在本实验中用到它。

1. 在 Azure 门户中，浏览到 **“az104-02-mg1”** 管理组并显示其 **“详细信息”**。

1. 依次单击 **“访问控制(IAM)”**，再单击 **“+ 添加”**，**“角色分配”**，然后将 **“支持请求参与者(自定义)”** 角色分配给新创建的用户帐户。

1. 打开一个 **“InPrivate”** 浏览器窗口，并使用新创建的用户帐户登录到 [Azure 门户](https://portal.azure.com)。当提示你更新密码时，请为用户更改密码。

    >**说明**：你无需键入用户名，而是可以直接粘贴剪贴板的内容。

1. 在 **“InPrivate”** 浏览器窗口，在 Azure 门户搜索并选择 **“资源组”** 以验证 az104-02-aaduser1 用户可以看到所有资源组。

1. 在 **“InPrivate”** 浏览器窗口，在 Azure 门户搜索并选择 **“全部资源”** 以验证 az104-02-aaduser1 用户看不到任何资源。

1. 在 **“InPrivate”** 浏览器窗口，在 Azure 门户搜索并选择 **“帮助和支持”**，然后单击 **“+ 新建支持请求”**。 

1. 在 **“InPrivate”** 浏览器窗口，在 **“帮助与支持 - 新的支持请求”** 边栏选项卡的 **“基本”** 选项卡上，选择 **“服务和订阅限制(配额)”** 问题类型，并请注意，你在本实验室中所使用的订阅已列在 **“订阅”** 下拉列表中。

    >**说明**： **“订阅”** 下拉列表中出现你在本实验室中所使用的订阅，表示你所使用的帐户具有创建特定于订阅的支持请求所需的权限。

    >**说明**：如果没有 **“服务与订阅限制(配额)”** 选项，从 Azure 门户注销并重新登录。

1. 不要继续创建支持请求。相反，请以“az104-02-aaduser1”用户身份从 Azure 门户注销，然后关闭 InPrivate 浏览器窗口。

#### 清理资源

   >**说明**：请记得移除任何你不再使用的新创建的 Azure 资源。 

   >**说明**：虽然在此实验室中创建的资源不会产生额外的费用，但请删除未使用的资源，以确保不会有意外费用产生。

1. 在 Azure 门户中，搜索并选择 **“Azure Active Directory”**，在“Azure Active Directory”边栏选项卡上，单击 **“用户”**。

1. 在 **“用户-所有用户”** 边栏选项卡中，单击 **“az104-02-aaduser1”**。

1. 在 **“az104-02-aaduser1 - 配置文件”** 边栏选项卡中，复制 **“对象 ID”** 属性值。

1. 在 Azure 门户中，在 **“Cloud Shell”** 中启动 **“PowerShell”** 会话。

1. 在“Cloud Shell”窗格中，运行以下命令以删除自定义角色定义的分配（将 `[object_ID]` 占位符替换为你在此任务前面复制的 **“az104-02-aaduser1”** Azure Active Directory 用户帐户的 **“对象 ID”** 属性的值）：

   ```pwsh
   $scope = (Get-AzRoleAssignment -RoleDefinitionName 'Support Request Contributor (Custom)').Scope

   Remove-AzRoleAssignment -ObjectId '[object_ID]' -RoleDefinitionName 'Support Request Contributor (Custom)' -Scope $scope
   ```

1. 在 Cloud Shell 窗格中，运行下列命令以删除自定义角色定义：

   ```pwsh
   Remove-AzRoleDefinition -Name 'Support Request Contributor (Custom)' -Force
   ```

1. 在 Azure 门户中，返回 **“Azure Active Directory ”** 的 **“用户 - 所有用户”** 边栏选项卡 ，然后删除 **“az104-02-aaduser1”** 用户帐户。

1. 在 Azure 门户中，导航到 **“az104-02-mg1”** 管理组并显示其 **“详细信息”**。

1. 右键单击代表你的 Azure 订阅的条目右侧的 **“省略号”** 图标，然后单击 **“移动”**。

1. 在 **“移动”** 边栏选项卡中，选择订阅最初所属的管理组，然后单击 **“保存”**。 

1. 导航回 **“管理组”** 边栏选项卡，右键单击 **“az104-02-mg1”** 管理组右侧的 **“省略号”** 图标，然后单击 **“删除”**。

#### 回顾

在本实验室中，你已：

- 实现管理组
- 创建自定义 RBAC 角色 
- 分配 RBAC 角色
