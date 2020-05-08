---
lab:
    title: '03c - 使用 Azure PowerShell 管理 Azure 资源'
    module: '模块 03 - Azure 管理'
---

# 实验室 03c - 使用 Azure PowerShell 管理 Azure 资源
# 学生实验室手册

## 实验室场景

你已经探索了与通过使用 Azure 门户和 Azure 资源管理器模板基于资源组对资源进行预配和组织相关的基本 Azure 管理功能，现在需要使用 Azure PowerShell 执行同等的任务。为避免安装 Azure PowerShell 模块，你将利用 Azure Cloud Shell 中可用的 PowerShell 环境。

## 目标

在本实验室中，你将：

+ 任务 1：在 Azure Cloud Shell 中启动一个 PowerShell 会话
+ 任务 2：使用 Azure PowerShell 创建一个资源组和一个 Azure 托管磁盘
+ 任务 3：使用 Azure PowerShell 配置托管磁盘

## 预计用时：20 分钟

## 说明

### 练习 1：

#### 任务 1：在 Azure Cloud Shell 中启动一个 PowerShell 会话。

在此任务中，你将在 Cloud Shell 中打开一个 PowerShell 会话。 

1. 在门户中，如需打开 **“Azure Cloud Shell”**，可单击 Azure 门户右上角的“Azure Cloud Shell”图标。

1. 如果提示需要选择 **“Bash”** 或 **“PowerShell”**，请选择 **“PowerShell”**。 

    >**说明**：如果这是你第一次启动 **“Cloud Shell”**，并看到 **“你没有装载存储”** 的消息，请选择你在本实验室中使用的订阅，然后单击 **“创建存储”**。 

1. 如果出现提示，请单击 **“创建存储”**，然后等待直到出现“Azure Cloud Shell”窗格。 

1. 确保 **“PowerShell”** 出现在“Cloud Shell”窗格左上角的下拉菜单中。

#### 任务 2：使用 Azure PowerShell 创建一个资源组和一个 Azure 托管磁盘

在此任务中，你将在 Cloud Shell 中使用 Azure PowerShell 会话来创建资源组和 Azure 托管磁盘

1. 要在 Cloud Shell 的 PowerShell 会话中创建一个资源组，该资源组与你在上一个实验中创建的资源组 **az104-03b-rg1** 位于同一个 Azure 区域，请运行以下命令：

   ```pwsh
   $location = (Get-AzResourceGroup -Name az104-03b-rg1).Location

   $rgName = 'az104-03c-rg1'

   New-AzResourceGroup -Name $rgName -Location $location
   ```
1. 若要检索新创建资源组的属性，请运行以下命令：

   ```pwsh
   Get-AzResourceGroup -Name $rgName
   ```
1. 要创建一个新的托管磁盘，并使该磁盘与你在本模块此前的实验室中创建的磁盘具有相同特征，请运行以下命令：

   ```pwsh
   $diskConfig = New-AzDiskConfig `
    -Location $location `
    -CreateOption Empty `
    -DiskSizeGB 32 `
    -Sku Standard_LRS

   $diskName = 'az104-03c-disk1'

   New-AzDisk `
    -ResourceGroupName $rgName `
    -DiskName $diskName `
    -Disk $diskConfig
   ```

1. 要检索新建磁盘的属性，请运行以下命令：

   ```pwsh
   Get-AzDisk -ResourceGroupName $rgName -Name $diskName
   ```

#### 任务 3：使用 Azure PowerShell 配置托管磁盘

在此任务中，你将使用 Cloud Shell 中的 Azure PowerShell 会话来管理 Azure 托管磁盘的配置。 

1. 若要在 Cloud Shell 中的 PowerShell 会话中将 Azure 托管磁盘的大小增加到 **64 GB**，请运行以下命令：

   ```pwsh
   New-AzDiskUpdateConfig -DiskSizeGB 64 | Update-AzDisk -ResourceGroupName $rgName -DiskName $diskName
   ```

1. 要验证更改是否生效，请运行以下命令：

   ```pwsh
   Get-AzDisk -ResourceGroupName $rgName -Name $diskName
   ```

1. 要验证当前 SKU 是否为**Standard_LRS**，请运行以下命令：

   ```pwsh
   (Get-AzDisk -ResourceGroupName $rgName -Name $diskName).Sku
   ```

1. 如要在 Cloud Shell 中的 PowerShell 会话中将磁盘性能 SKU 更改为 **“Premium_LRS”**，运行以下命令：

   ```pwsh
   New-AzDiskUpdateConfig -Sku Premium_LRS | Update-AzDisk -ResourceGroupName $rgName -DiskName $diskName
   ```

1. 要验证更改是否生效，请运行以下命令：

   ```pwsh
   (Get-AzDisk -ResourceGroupName $rgName -Name $diskName).Sku
   ```

#### 清理资源

   >**说明**：不要删除你在此实验室中部署的资源。在本实验室的下一个模块中，你将需要引用他们。

#### 回顾

在本实验室中，你已：

- 在 Azure Cloud Shell 中启动 PowerShell 会话
- 使用 Azure PowerShell 创建一个资源组和一个 Azure 托管磁盘
- 使用 Azure PowerShell 配置托管磁盘