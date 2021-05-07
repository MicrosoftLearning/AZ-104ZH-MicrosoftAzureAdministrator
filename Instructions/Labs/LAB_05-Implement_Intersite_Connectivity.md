---
lab:
    title: '05－实施网站间的连接性'
    module: '模块 05－网站间的连接性'
---

# 实验室 05 - 实现网站间的连接性
# 学生实验室手册

## 实验室场景

Contoso 在波士顿、纽约和西雅图办公室的数据中心通过网状广域网链接进行连接，彼此之间完全相连。需要实现一个能够反映 Contoso 本地网络拓扑并验证其功能的实验室环境

## 目标

在本实验室中，你将：

+ 任务 1：预配实验室环境
+ 任务 2：配置本地和全局虚拟网络对等互连
+ 任务 3：测试网站间的连接性

## 预计用时：30 分钟

### 说明

#### 任务 1：预配实验室环境

在此任务中，你将部署三个虚拟机，每个虚拟机都部署到一个独立的虚拟网络中。其中两个在同一个 Azure 区域中，第三个在另一个 Azure 区域中。

1. 登录到 [Azure 门户](https://portal.azure.com)。

1. 在 Azure 门户中，单击 Azure 门户右上方的图标，打开 **Azure Cloud Shell**。

1. 提示选择 **Bash** 或 **PowerShell** 时，选择 **PowerShell**。

    >**备注**：如果这是第一次启动 **Cloud Shell**，并看到 **“未装载任何存储”** 消息，请选择在本实验室中使用的订阅，然后选择 **“创建存储”**。

1. 在 Cloud Shell 窗格的工具栏中，单击 **“上传/下载文件”** 图标，在下拉菜单中单击 **“上传”**，然后将文件 **\\Allfiles\\Labs\\05\\az104-05-vnetvm-loop-template.json** 和 **\\Allfiles\\Labs\\05\\az104-05-vnetvm-loop-parameters.json** 上传到 Cloud Shell 主目录中。

1. 在 Cloud Shell 窗格中，运行以下命令以创建将托管实验室环境的资源组。前两个虚拟网络和一对虚拟机将部署在 `[Azure_region_1]` 中。第三个虚拟网络和第三个虚拟机将部署在同一资源组 `[Azure_region_2]` 中。（将占位符 `[Azure_region_1]` 和 `[Azure_region_2]` 替换为你打算在其中部署这些 Azure 虚拟机的两个不同 Azure 区域的名称）：

   ```powershell
   $location1 = '[Azure_region_1]'

   $location2 = '[Azure_region_2]'

   $rgName = 'az104-05-rg1'

   New-AzResourceGroup -Name $rgName -Location $location1
   ```

   >**备注**：为了识别 Azure 区域，请从 Cloud Shell 中的 PowerShell 会话中运行 **(Get-AzLocation).Location**

1. 在 Cloud Shell 窗格中，运行以下命令创建三个虚拟网络，并使用上传的模板和参数文件将虚拟机部署到其中：

   ```powershell
   New-AzResourceGroupDeployment `
      -ResourceGroupName $rgName `
      -TemplateFile $HOME/az104-05-vnetvm-loop-template.json `
      -TemplateParameterFile $HOME/az104-05-vnetvm-loop-parameters.json `
      -location1 $location1 `
      -location2 $location2
   ```

    >**备注**：请等待部署完成之后再继续下一步。该操作需约 2 分钟。

1. 关闭 Cloud Shell 窗格。

#### 任务 2：配置本地和全局虚拟网络对等互连

在此任务中，你需要在之前的任务中部署的虚拟网络之间配置本地和全局对等互连。

1. 在 Azure 门户中，搜索并选择 **“虚拟网络”**。

1. 查看你在上一个任务中创建的虚拟网络，并验证是否前两个位于相同的 Azure 区域、第三个位于不同的 Azure 区域。

    >**备注**：用于部署三个虚拟网络的模板可确保三个虚拟网络的 IP 地址范围不重叠。

1. 在虚拟网络列表中，单击 **“az104-05-vnet0”**。

1. 在 **“az104-05-vnet0”** 虚拟网络边栏选项卡的 **“设置”** 部分，单击 **“对等互连”**，然后单击 **“+ 添加”**。

1. 添加一个对等互连，设置如下（其他设置保留默认值），然后单击 **“添加”**：

    | 设置 | 值|
    | --- | --- |
    | 此虚拟网络：对等互连链接名称 | **az104-05-vnet0_to_az104-05-vnet1** |
    | 此虚拟网络：流向远程虚拟网络的流量 | **允许（默认）** |
    | 此虚拟网络：从远程虚拟网络转发的流量 | **阻止来自此虚拟网络外部的流量** |
    | 虚拟网络网关 | **无** |
    | 远程虚拟网络：对等互连链接名称 | **az104-05-vnet1_to_az104-05-vnet0** |
    | 虚拟网络部署模型 | **资源管理器** |
    | 我知道我的资源 ID | 未选定 |
    | 订阅 | 在本实验室中使用的 Azure 订阅的名称 |
    | 虚拟网络 | **az104-05-vnet1** |
    | 流向远程虚拟网络的流量 | **允许（默认）** |
    | 从远程虚拟网络转发的流量 | **阻止来自此虚拟网络外部的流量** |
    | 虚拟网络网关 | **无** |

    >**备注**：此步骤将建立两个本地对等互连 - 一个从 az104-05-vnet0 到 az104-05-vnet1，另一个从 az104-05-vnet1 到 az104-05-vnet0。

    >**备注**： 如果 Azure 门户界面未显示上一任务中创建的虚拟网络，可以在 Cloud Shell 中运行以下 PowerShell 命令来配置对等互连：
    
   ```powershell
   $rgName = 'az104-05-rg1'

   $vnet0 = Get-AzVirtualNetwork -Name 'az104-05-vnet0' -ResourceGroupName $rgname

   $vnet1 = Get-AzVirtualNetwork -Name 'az104-05-vnet1' -ResourceGroupName $rgname

   Add-AzVirtualNetworkPeering -Name 'az104-05-vnet0_to_az104-05-vnet1' -VirtualNetwork $vnet0 -RemoteVirtualNetworkId $vnet1.Id

   Add-AzVirtualNetworkPeering -Name 'az104-05-vnet1_to_az104-05-vnet0' -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $vnet0.Id
   ``` 

1. 在 **“az104-05-vnet0”** 虚拟网络边栏选项卡的 **“设置”** 部分，单击 **“对等互连”**，然后单击 **“+ 添加”**。

1. 添加一个对等互连，设置如下（其他设置保留默认值），然后单击 **“添加”**：

    | 设置 | 值|
    | --- | --- |
    | 此虚拟网络：对等互连链接名称 | **az104-05-vnet0_to_az104-05-vnet2** |
    | 此虚拟网络：流向远程虚拟网络的流量 | **允许（默认）** |
    | 此虚拟网络：从远程虚拟网络转发的流量 | **阻止来自此虚拟网络外部的流量** |
    | 虚拟网络网关 | **无** |
    | 远程虚拟网络：对等互连链接名称 | **az104-05-vnet2_to_az104-05-vnet0** |
    | 虚拟网络部署模型 | **资源管理器** |
    | 我知道我的资源 ID | 未选定 |
    | 订阅 | 在本实验室中使用的 Azure 订阅的名称 |
    | 虚拟网络 | **az104-05-vnet2** |
    | 流向远程虚拟网络的流量 | **允许（默认）** |
    | 从远程虚拟网络转发的流量 | **阻止来自此虚拟网络外部的流量** |
    | 虚拟网络网关 | **无** |

    >**备注**：此步骤建立两个全局对等互连：一个从 az104-05-vnet0 到 az104-05-vnet2，另一个从 az104-05-vnet2 到 az104-05-vnet0。

    >**备注**： 如果 Azure 门户界面未显示上一任务中创建的虚拟网络，可以在 Cloud Shell 中运行以下 PowerShell 命令来配置对等互连：
    
   ```powershell
   $rgName = 'az104-05-rg1'

   $vnet0 = Get-AzVirtualNetwork -Name 'az104-05-vnet0' -ResourceGroupName $rgname

   $vnet2 = Get-AzVirtualNetwork -Name 'az104-05-vnet2' -ResourceGroupName $rgname

   Add-AzVirtualNetworkPeering -Name 'az104-05-vnet0_to_az104-05-vnet2' -VirtualNetwork $vnet0 -RemoteVirtualNetworkId $vnet2.Id

   Add-AzVirtualNetworkPeering -Name 'az104-05-vnet2_to_az104-05-vnet0' -VirtualNetwork $vnet2 -RemoteVirtualNetworkId $vnet0.Id
   ``` 

1. 导航回 **“虚拟网络”** 边栏选项卡，然后在虚拟网络列表中单击 **“az104-05-vnet1”**。

1. 在 **“az104-05-vnet1”** 虚拟网络边栏选项卡的 **“设定”** 部分，单击 **“对等互连”**，然后单击 **“+ 添加”**。

1. 添加一个对等互连，设置如下（其他设置保留默认值），然后单击 **“添加”**：

    | 设置 | 值|
    | --- | --- |
    | 此虚拟网络：对等互连链接名称 | **az104-05-vnet1_to_az104-05-vnet2** |
    | 此虚拟网络：流向远程虚拟网络的流量 | **允许（默认）** |
    | 此虚拟网络：从远程虚拟网络转发的流量 | **阻止来自此虚拟网络外部的流量** |
    | 虚拟网络网关 | **无** |
    | 远程虚拟网络：对等互连链接名称 | **az104-05-vnet2_to_az104-05-vnet1** |
    | 虚拟网络部署模型 | **资源管理器** |
    | 我知道我的资源 ID | 未选定 |
    | 订阅 | 在本实验室中使用的 Azure 订阅的名称 |
    | 虚拟网络 | **az104-05-vnet2** |
    | 流向远程虚拟网络的流量 | **允许（默认）** |
    | 从远程虚拟网络转发的流量 | **阻止来自此虚拟网络外部的流量** |
    | 虚拟网络网关 | **无** |

    >**备注**：此步骤建立两个全局对等互连：一个从 az104-05-vnet1 到 az104-05-vnet2，另一个从 az104-05-vnet2 到 az104-05-vnet1。

    >**备注**： 如果 Azure 门户界面未显示上一任务中创建的虚拟网络，可以在 Cloud Shell 中运行以下 PowerShell 命令来配置对等互连：
    
   ```powershell
   $rgName = 'az104-05-rg1'

   $vnet1 = Get-AzVirtualNetwork -Name 'az104-05-vnet1' -ResourceGroupName $rgname

   $vnet2 = Get-AzVirtualNetwork -Name 'az104-05-vnet2' -ResourceGroupName $rgname

   Add-AzVirtualNetworkPeering -Name 'az104-05-vnet1_to_az104-05-vnet2' -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $vnet2.Id

   Add-AzVirtualNetworkPeering -Name 'az104-05-vnet2_to_az104-05-vnet1' -VirtualNetwork $vnet2 -RemoteVirtualNetworkId $vnet1.Id
   ``` 

#### 任务 3：测试网站间的连接性

在此任务中，将测试在上一个任务中通过本地和全局对等互连连接的三个虚拟网络上的虚拟机之间的连接性。

1. 在 Azure 门户中，搜索并选择 **“虚拟机”**。

1. 在虚拟机列表中，单击 **“az104-05-vm0”**。

1. 在 **“az104-05-vm0”** 边栏选项卡中，单击 **“连接”**，在下拉菜单中，单击 **“RDP”**，在 **“连接到 RDP”** 边栏选项卡中，单击 **“下载 RDP 文件”**，并按照提示启动远程桌面会话。

    >**备注**：此步骤是指在 Windows 计算机中通过远程桌面进行连接。在 Mac 上，可以使用 Mac App Store 中的远程桌面客户端，而在 Linux 计算机上，可以使用开源 RDP 客户端软件。

    >**备注**：连接到目标虚拟机时，可以忽略任何警告提示。

1. 出现提示时，请使用 **“Student”** 用户名和 **“Pa55w.rd1234”** 密码登录。

1. 在与 **az104-05-vm0** 的远程桌面会话中，右键单击 **“开始”** 按钮，然后在右键菜单中单击 **“Windows PowerShell (管理员)”**。

1. 在 Windows PowerShell 控制台窗口，运行以下命令以测试通过 TCP 端口 3389 与 **az104-05-vm1**（其专用 IP 地址为 **10.51.0.4**）的连接性：

   ```powershell
   Test-NetConnection -ComputerName 10.51.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```

    >**备注**：该测试使用 TCP 3389，因为这是操作系统防火墙默认允许使用的端口。

1. 检查命令输出并验证连接是否成功。

1. 在 Windows PowerShell 控制台窗口中，运行以下命令以测试与 **az104-05-vm2**（其专用 IP 地址为 **10.52.0.4**）的连接性：

   ```powershell
   Test-NetConnection -ComputerName 10.52.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```

1. 在实验室计算机上切换回 Azure 门户，然后返回 **“虚拟机”** 边栏选项卡。

1. 在虚拟机列表中，单击 **“az104-05-vm1”**。

1. 在 **“az104-05-vm1”** 边栏选项卡中，单击 **“连接”**，在下拉菜单中，单击 **“RDP”**，在 **“连接到 RDP”** 边栏选项卡中，单击 **“下载 RDP 文件”**，并按照提示启动远程桌面会话。

    >**备注**：此步骤是指在 Windows 计算机中通过远程桌面进行连接。在 Mac 上，可以使用 Mac App Store 中的远程桌面客户端，而在 Linux 计算机上，可以使用开源 RDP 客户端软件。

    >**备注**：连接到目标虚拟机时，可以忽略任何警告提示。

1. 出现提示时，请使用 **“Student”** 用户名和 **“Pa55w.rd1234”** 密码登录。

1. 在与 **az104-05-vm1** 的远程桌面会话中，右键单击 **“开始”** 按钮，然后在右键菜单中单击 **“Windows PowerShell (管理员)”**。

1. 在 Windows PowerShell 控制台窗口，运行以下命令以测试通过 TCP 端口 3389 与 **az104-05-vm2**（其专用 IP 地址为 **10.52.0.4**）的连接性：

   ```powershell
   Test-NetConnection -ComputerName 10.52.0.4 -Port 3389 -InformationLevel 'Detailed'
   ```

    >**备注**：该测试使用 TCP 3389，因为这是操作系统防火墙默认允许使用的端口。

1. 检查命令输出并验证连接是否成功。

#### 清理资源

   >**备注**：请记得删除任何新创建而不会再使用的 Azure 资源。删除未使用的资源，确保不产生意外费用。

1. 在 Azure 门户中，在 **Cloud Shell** 窗格中打开 **“PowerShell”** 会话。

1. 运行以下命令，列出在本模块各实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'az104-05*'
   ```

1. 运行以下命令，删除在本模块各个实验室中创建的所有资源组：

   ```powershell
   Get-AzResourceGroup -Name 'az104-05*' | Remove-AzResourceGroup -Force -AsJob
   ```

    >**备注**：该命令以异步方式执行（由 -AsJob 参数决定），因此，虽然你随后可在同一 PowerShell 会话中立即运行另一个 PowerShell 命令，但实际上要花几分钟才能删除资源组。

#### 回顾

在本实验室中，你已：

+ 预配实验室环境
+ 配置本地和全局虚拟网络对等互连
+ 测试网站间的连接性
