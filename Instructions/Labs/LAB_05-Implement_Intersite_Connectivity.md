---
lab:
    title: '05－实现站点间连接'
    module: '模块 05－站点间连接'
---

# 实验室 05－实现站点间连接
# 学生实验室手册

## 实验室场景

Contoso 在波士顿、纽约和西雅图的办公室设有数据中心。它们通过网状广域网链路进行连接，相互之间可以完全连接。你需要实现一个实验室环境，以反映 Contoso 本地网络的拓扑并验证其功能。 

## 目标

在本实验室中，你将：

+ 任务 1：预配实验室环境
+ 任务 2：配置本地和全局虚拟网络对等互连
+ 任务 3：测试站点间连接 

## 预计用时：30 分钟

### 说明

#### 任务 1：预配实验室环境

在此任务中，你将部署三个虚拟机，每个虚拟机都部署到一个独立的虚拟网络中。其中两个在同一个 Azure 区域中，第三个在另一个 Azure 区域中。 

1. 登录到 [Azure 门户](https://portal.azure.com)。

1. 在 Azure 门户中，单击 Azure 门户右上角的图标打开 **“Azure Cloud Shell”**。

1. 如果提示需要选择 **“Bash”** 或 **“PowerShell”**，请选择 **“PowerShell”**。 

    >**说明**：如果这是你第一次启动 **“Cloud Shell”**，并看到 **“你没有装载存储”** 的消息，请选择你在本实验中使用的订阅，然后单击 **“创建存储”**。 

1. 在“Cloud Shell”窗格的工具栏中，单击 **“上传/下载文件”** 图标，在下拉菜单中，单击 **“上传”** 并将文件 **“\\Allfiles\\Labs\\05\\az104-05-vnetvm-template.json”** 和 **“\\Allfiles\\Labs\\05\\az104-05-vnetvm-parameters.json”** 上传到 Cloud Shell 主目录中。

1. 在“Cloud Shell”窗格中，运行以下命令创建第一个资源组，用于托管第一个虚拟网络和虚拟机对（在打算部署 Azure 虚拟机的地方，用 Azure 区域名称替换`[Azure_region_1]`占位符）：

   ```pwsh
   $location = '[Azure_region_1]'

   $rgName = 'az104-05-rg0'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. 在“Cloud Shell”窗格中，运行以下命令以创建第一个虚拟网络，并使用你上传的模板和参数文件将一台虚拟机部署到其中：

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 0 `
      -AsJob
   ```
1. 在“Cloud Shell”窗格中，运行以下命令创建第二个资源组，该资源组将托管第二个虚拟网络和第二台虚拟机。

   ```pwsh
   $rgName = 'az104-05-rg1'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. 在“Cloud Shell”窗格中，运行以下命令来创建第二个虚拟网络，并使用你上传的模板和参数文件将一台虚拟机部署到其中：

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 1 `
      -AsJob
   ```
1. 在“Cloud Shell”窗格中，运行以下命令创建第三个资源组，该资源组将用于托管第三个虚拟网络和第三台虚拟机（将 `[Azure_region_2]` 占位符替换为另一个要部署 Azure 虚拟机的 Azure 区域名称，该区域要与你用于其他两个部署的 Azure 区域不同。）

   ```pwsh
   $location = '[Azure_region_2]'

   $rgName = 'az104-05-rg2'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. 在“Cloud Shell”窗格中，运行以下命令，以创建第三个虚拟网络，并使用上传的模板和参数文件将一台虚拟机部署到其中：

   ```pwsh
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-parameters.json `
      -nameSuffix 2 `
      -AsJob
   ```
    >**注意**：等待部署完成，再执行下一个任务。该操作需约 2 分钟。

    >**说明**：如要验证部署状态，你可检查在此任务中所创建资源组的属性。

1. 关闭 Cloud Shell 窗格。

#### 任务 2：配置本地和全局虚拟网络对等互连

在此任务中，你需要在之前的任务中部署的虚拟网络之间配置本地和全局对等互连。

1. 在 Azure 门户中，搜索并选择 **“虚拟网络”**。

1. 查看你在上一个任务中创建的虚拟网络，并验证前两个是否位于相同的 Azure 区域中，且第三个位于不同的 Azure 区域中。 

    >**说明**：用于部署三个虚拟网络的模板可确保三个虚拟网络的 IP 地址范围不重叠。

1. 在虚拟网络列表中，单击 **“az104-05-vnet0”**。

1. 在 **“az104-05-vnet0”** 虚拟网络边栏选项卡的 **“设定”** 部分单击 **“对等互连”**，然后单击 **“+ 添加”**。

1. 添加一个对等互连，设置如下（其他设置则保留默认值）：

    | 设置 | 数值|
    | --- | --- |
    | 从“az104-05-vnet0”到“远程虚拟网络”的对等互连名称 | **az104-05-vnet0_to_az104-05-vnet1** |
    | 虚拟网络部署模型 | **资源管理器** |
    | 订阅 | 你在本实验中使用的 Azure 订阅的名称 |
    | 虚拟网络 | **az104-05-vnet1 (az104-05-rg1)** |
    | 从 az104-05-vnet1 到 az104-05-vnet0 的对等互联名称 | **az104-05-vnet1_to_az104-05-vnet0** |
    | 允许从 az104-05-vnet0 到 az104-05-vnet1 的虚拟网络访问 | **已启用** |
    | 允许从 az104-05-vnet1 到 az104-05-vnet0 的虚拟网络访问 | **已启用** |
    | 允许从 az104-05-vnet1 转发流量到 az104-05-vnet0 | **禁用** |
    | 允许从 az104-05-vnet0 转发流量到 az104-05-vnet1 | **禁用** |
    | 允许网关传输 | **（取消勾选复选框）** |

    >**说明**：此步骤将建立两个本地对等互连 - 一个从 az104-05-vnet0 到 az104-05-vnet1，另一个从 az104-05-vnet1 到 az104-05-vnet0。

1. 在 **“az104-05-vnet0”** 虚拟网络边栏选项卡的 **“设定”** 部分单击 **“对等互连”**，然后单击 **“+ 添加”**。

1. 添加一个对等互连，设置如下（其他设置则保留默认值）：

    | 设置 | 数值|
    | --- | --- |
    | 从“az104-05-vnet0”到“远程虚拟网络”的对等互连名称 | **az104-05-vnet0_to_az104-05-vnet2** |
    | 虚拟网络部署模型 | **资源管理器** |
    | 订阅 | 你在本实验中使用的 Azure 订阅的名称 |
    | 虚拟网络 | **az104-05-vnet2 (az104-05-rg2)** |
    | 从 az104-05-vnet2 到 az104-05-vnet0 的对等互连名称 | **az104-05-vnet2_to_az104-05-vnet0** |
    | 允许从 az104-05-vnet0 到 az104-05-vnet2 的虚拟网络访问 | **已启用** |
    | 允许从 az104-05-vnet2 到 az104-05-vnet0 的虚拟网络访问 | **已启用** |
    | 允许从 az104-05-vnet2 转发流量到 az104-05-vnet0 | **禁用** |
    | 允许从 az104-05-vnet0 转发流量到 az104-05-vnet2 | **禁用** |
    | 允许网关传输 | **（取消勾选复选框）** |

    >**说明**：此步骤建立两个全局互连：一个从 az104-05-vnet0 到 az104-05-vnet2，另一个从 az104-05-vnet2 到 az104-05-vnet0。

1. 导航回 **“虚拟网络”** 边栏选项卡，然后在虚拟网络列表中单击 **az104-05-vnet1**。

1. 在 **“az104-05-vnet1”** 虚拟网络边栏选项卡中，在 **“设置”** 部分，单击 **“对等互连”**，然后单击 **“+ 添加”**。

1. 添加一个对等互连，设置如下（其他设置则保留默认值）：

    | 设置 | 数值|
    | --- | --- |
    | 从 az104-05-vnet1 到远程虚拟网络的对等互连名称 | **az104-05-vnet1_to_az104-05-vnet2** |
    | 虚拟网络部署模型 | **资源管理器** |
    | 订阅 | 你在本实验中使用的 Azure 订阅的名称 |
    | 虚拟网络 | **az104-05-vnet2 (az104-05-rg2)** |
    | 从 az104-05-vnet2 到 az104-05-vnet1 的对等互连的名称 | **az104-05-vnet2_to_az104-05-vnet1** |
    | 允许从 az104-05-vnet1 到 az104-05-vnet2 的虚拟网络访问 | **已启用** |
    | 允许从 az104-05-vnet2 到 az104-05-vnet1 的虚拟网络访问 | **已启用** |
    | 允许从 az104-05-vnet2 转发流量到 az104-05-vnet1 | **禁用** |
    | 允许从 az104-05-vnet1 转发流量到 az104-05-vnet2 | **禁用** |
    | 允许网关传输 | **（取消勾选复选框）** |

    >**说明**：此步骤建立了两个全局对等互连 - 一个从 az104-05-vnet1 到 az104-05-vnet2，另一个从 az104-05-vnet2 到 az104-05-vnet1。

#### 任务 3：测试站点间连接 

在此任务中，将测试在上一个任务中通过本地和全局对等互连连接的三个虚拟网络上的虚拟机之间的连接性。

1. 在 Azure 门户中，搜索并选择 **“虚拟机”**。

1. 在虚拟机列表中，单击 **“az104-05-vm0”**。

1. 在 **“az104-05-vm0”** 边栏选项卡中，单击 **“连接”**，在下拉菜单中，单击 **“RDP”**，在 **“与 RDP 连接”** 边栏选项卡中，单击 **“下载 RDP 文件”** 并按照提示启动远程桌面会话。

    >**说明**：此步骤参阅通过远程桌面与 Windows 计算机进行连接。在 Mac 上，你可以从 Mac 应用商店下载远程桌面客户端；而在 Linux 计算机上，你可以使用开放源代码 RDP 客户端软件。

    >**说明**：当连接到目标虚拟机时，你可以忽略任何警告提示。

1. 当出现提示时，请使用用户名 Student 和密码 **Pa55w.rd1234** 登录。

1. 在与 **az104-05-vm0** 的远程桌面会话中，右键单击 **“开始”** 按钮，然后在右键菜单中单击 **“Windows PowerShell（管理员）”**。

1. 在 Windows PowerShell 控制台窗口，运行以下命令以测试通过 TCP 端口 3389 与 **az104-05-vm1** （其专用 IP 地址为 **10.51.0.4**）的连接性：

   ```pwsh
   Test-NetConnection -ComputerName 10.51.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
    >**注意**：该测试使用 TCP 3389，因为这是操作系统防火墙默认允许使用的端口。 

1. 检查命令输出并验证连接是否成功。

1. 在 Windows PowerShell 控制台窗口中，运行以下命令以测试与 **az104-05-vm2** （其专用 IP 地址为 **10.52.0.4**）的连接性：

   ```pwsh
   Test-NetConnection -ComputerName 10.52.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
1. 切换回实验室计算机上的 Azure 门户，然后返回 **“虚拟机”** 边栏选项卡。 

1. 在虚拟机列表中，单击 **az104-05-vm1**。

1. 在 **az104-05-vm1** 边栏选项卡，单击 **“连接”**，在下拉菜单中，单击 **“RDP”**，在 **“与 RDP 连接”** 边栏选项卡，单击 **“下载 RDP 文件”** 并按照提示启动远程桌面会话。

    >**说明**：此步骤参阅通过远程桌面与 Windows 计算机进行连接。在 Mac 上，你可以从 Mac 应用商店下载远程桌面客户端；而在 Linux 计算机上，你可以使用开放源代码 RDP 客户端软件。

    >**说明**：当连接到目标虚拟机时，你可以忽略任何警告提示。

1. 当出现提示时，请使用用户名 Student 和密码 **Pa55w.rd1234** 登录。

1. 在与 **az104-05-vm1** 的远程桌面会话中，右键单击 **“开始”** 按钮，然后在右键菜单中单击 **“Windows PowerShell（管理员）”**。

1. 在 Windows PowerShell 控制台窗口中，运行以下命令以测试 **az104-05-vm2** （其专用 IP 地址为 **10.52.0.4**）在 TCP 端口 3389 的连接性：

   ```pwsh
   Test-NetConnection -ComputerName 10.52.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```
    >**注意**：该测试使用 TCP 3389，因为这是操作系统防火墙默认允许使用的端口。 

1. 检查命令输出并验证连接是否成功。

#### 清理资源

   >**说明**：请记得移除任何你不再使用的新创建的 Azure 资源。移除未使用的资源，确保不产生意外费用。

1. 在 Azure 门户中，打开 **“Cloud Shell”** 窗格中的 **“PowerShell”** 会话。

1. 通过运行以下命令，列出在本模块实验室中创建的所有资源组：

   ```pwsh
   Get-AzResourceGroup -Name 'az104-05*'
   ```

1. 通过运行以下命令，删除在本模块实验室中创建的所有资源组：

   ```pwsh
   Get-AzResourceGroup -Name 'az104-05*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**注意**：该命令异步执行（由 -AsJob 参数确定），因此尽管此后你可以立即在同一 PowerShell 会话中运行另一个 PowerShell 命令，但实际上要花几分钟才能移除资源组。

#### 回顾

在本实验室中，你已：

- 预配实验室环境
- 配置本地和全局虚拟网络对等互连
- 测试站点间连接性 
