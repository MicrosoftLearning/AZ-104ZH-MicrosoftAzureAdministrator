---
lab:
    title: '03d - 使用 Azure CLI 管理 Azure 资源'
    module: '模块 03 - Azure 管理'
---

# 实验室 03d - 使用 Azure CLI 管理 Azure 资源
# 学生实验室手册

## 实验室场景

你已经探索了与使用 Azure 门户、Azure 资源管理器模板和 Azure PowerShell 预配资源并根据资源组整理资源相关的基本 Azure 管理功能，现在你需要使用 Azure CLI 执行相应的任务。为了避免安装 Azure CLI，你将利用 Azure Cloud Shell 中提供的 Bash 环境。

## 目标

在本实验室中，你将：

+ 任务 1： 在 Azure Cloud Shell 中启动 Bash 会话
+ 任务 2： 使用 Azure CLI 创建资源组和 Azure 托管磁盘
+ 任务 3： 使用 Azure CLI 配置托管磁盘

## 预计用时：20 分钟

## 说明

### 练习 1

#### 任务 1： 在 Azure Cloud Shell 中启动 Bash 会话

在此任务中，你将在 Cloud Shell 中打开 Bash 会话。 

1. 单击 Azure 门户右上方的图标，在门户中打开 **Azure Cloud Shell**。

1. 如果提示选择 **“Bash”** 或 **“PowerShell”**，请选择 **“Bash”**。 

    >**注意**：如果这是你首次启动 **Cloud Shell**，且看到 **“未装载任何存储”** 消息，请选择在本实验室中使用的订阅，然后选择 **“创建存储”**。 

1. 如果出现提示，请单击 **“创建存储”**，然后等到出现“Azure Cloud Shell”窗格。 

1. 确保 **“Bash”** 出现在 “Cloud Shell” 窗格左上角的下拉菜单中。

#### 任务 2： 使用 Azure CLI 创建资源组和 Azure 托管磁盘

在此任务中，你将使用 Cloud Shell 中的 Azure CLI 会话来创建资源组和 Azure 托管磁盘。

1. 要通过 Cloud Shell 中的 Bash 会话创建一个资源组，并且使该资源组与上一个实验室中创建的资源组 **az104-03c-rg1** 位于同一 Azure 区域，请运行以下命令：

   ```sh
   LOCATION=$(az group show --name 'az104-03c-rg1' --query location --out tsv)

   RGNAME='az104-03d-rg1'

   az group create --name $RGNAME --location $LOCATION
   ```
1. 要检索新创建资源组的属性，请运行以下命令：

   ```sh
   az group show --name $RGNAME
   ```
1. 要通过 Cloud Shell 中的 Bash 会话创建一个新的托管磁盘，并使该托管磁盘与你在本模块先前的实验室中创建的托管磁盘具有相同特征，请运行以下命令：

   ```sh
   DISKNAME='az104-03d-disk1'

   az disk create \
   --resource-group $RGNAME \
   --name $DISKNAME \
   --sku 'Standard_LRS' \
   --size-gb 32
   ```
    >**注意**：使用多行语法时，请确保每行都以反斜杠 (`\`) 结尾且没有尾随空格，并确保每行开头都没有前导空格。

1. 要检索新建磁盘的属性，请运行以下命令：

   ```sh
   az disk show --resource-group $RGNAME --name $DISKNAME
   ```

#### 任务 3： 使用 Azure CLI 配置托管磁盘

在此任务中，你将使用 Cloud Shell 中的 Azure CLI 会话来管理 Azure 托管磁盘的配置。 

1. 要通过 Cloud Shell 中的 Bash 会话将 Azure 托管磁盘的大小增加到 **64 GB**，请运行以下命令：

   ```sh
   az disk update --resource-group $RGNAME --name $DISKNAME --size-gb 64
   ```

1. 要验证更改是否生效，请运行以下命令：

   ```sh
   az disk show --resource-group $RGNAME --name $DISKNAME --query diskSizeGb
   ```

1. 要通过 Cloud Shell 中的 Bash 会话将磁盘性能 SKU 更改为 **“Premium_LRS”**，请运行以下命令：

   ```sh
   az disk update --resource-group $RGNAME --name $DISKNAME --sku 'Premium_LRS'
   ```

1. 要验证更改是否生效，请运行以下命令：

   ```sh
   az disk show --resource-group $RGNAME --name $DISKNAME --query sku
   ```

#### 清理资源

   >**注意**：请记得删除任何新创建而不会再使用的 Azure 资源。删除未使用的资源，确保不产生意外费用。

1. 在 Azure 门户中，在 **“Cloud Shell”** 窗格中打开 **“Bash”** shell 会话。

1. 运行以下命令，列出在本模块各实验室中创建的所有资源组：

   ```sh
   az group list --query "[?starts_with(name,'az104-03')].name" --output tsv
   ```

1. 运行以下命令，删除在本模块各个实验室中创建的所有资源组：

   ```sh
   az group list --query "[?starts_with(name,'az104-03')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

    >**注意**：该命令以异步方式执行（由 --nowait 参数决定），因此，虽然你随后可在同一 Bash 会话中立即运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

#### 回顾

在本实验室中，你已：

- 在 Azure Cloud Shell 中启动 Bash 会话
- 使用 Azure CLI 创建资源组和 Azure 托管磁盘
- 使用 Azure CLI 配置托管磁盘
