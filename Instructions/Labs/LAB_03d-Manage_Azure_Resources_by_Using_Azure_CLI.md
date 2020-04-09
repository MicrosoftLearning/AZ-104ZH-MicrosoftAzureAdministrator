---
lab:
    title: '03d - 使用 Azure CLI 管理Azure 资源'
    module: '模块 03 - Azure 管理'
---

# 实验室03d - 使用 Azure CLI 管理 Azure 资源
# 学生实验室手册

## 实验室场景

你已经探索了与通过使用 Azure 门户、Azure 资源管理器模板和 Azure PowerShell 基于资源组对资源进行预配和组织相关的基本 Azure 管理功能，现在需要使用 Azure CLI 执行同等的任务。为避免安装 Azure CLI，你将利用 Azure Cloud Shell 中可用的 Bash 环境。

## 目标

在本实验室中，你将：

+ 任务 1：在 Azure Cloud Shell 中启动 Bash 会话
+ 任务 2：使用 Azure CLI创建资源组和 Azure 托管磁盘
+ 任务 3：使用 Azure CLI 配置 托管磁盘

## 说明

### 练习 1：

#### 任务 1：在 Azure Cloud Shell 中启动 Bash 会话。

在此任务中，你需要在 Cloud Shell 中打开 Bash 会话。 

1. 单击 Azure 门户右上角的图标打开 **“Azure Cloud Shell”**。

1. 如果提示你选择 **“Bash”** 或者 **“PowerShell”**，选择 **“Bash”**。 

    >**说明**：如果这是你第一次启动 **“Cloud Shell”**，并看到 **“你没有装载存储”** 的消息，请选择你在本实验室中使用的订阅，然后单击 **“创建存储”**。 

1. 如果出现提示，请单击 **“创建存储”**，然后等待直到出现“Azure Cloud Shell”窗格。 

1. 确保 **“Bash”** 出现在“Cloud Shell”窗格左上角的下拉菜单中。

#### 任务 2：使用 Azure CLI 创建资源组和 Azure 托管磁盘

在此任务中，你将在 Cloud Shell 中使用 Azure CLI 会话来创建资源组和 Azure 托管磁盘。

1. 要在与你在上一个实验室中创建的 **az104-03c-rg1** 资源组相同的 Azure 区域中创建一个资源组，请在 Cloud Shell 中的 Bash 会话中运行下列命令：

   ```sh
   LOCATION=$(az group show --name 'az104-03c-rg1' --query location --out tsv)

   RGNAME='az104-03d-rg1'

   az group create --name $RGNAME --location $LOCATION
   ```
1. 若要检索新创建资源组的属性，请运行以下命令：

   ```sh
   az group show --name $RGNAME
   ```
1. 要创建一个新的托管磁盘，并使该磁盘与你在本模块此前的实验室中创建的磁盘具有相同特征，请从 Cloud Shell 中的 Bash 会话运行以下命令：

   ```sh
   DISKNAME='az104-03d-disk1'

   az disk create \
   --resource-group $RGNAME \
   --name $DISKNAME \
   --sku 'Standard_LRS' \
   --size-gb 32
   ```
    >**注意**：使用多行语法时，请确保每行以反斜杠 (`\`) 结尾，没有尾随空格，而且每行的开头都没有前导空格。

1. 要检索新建磁盘的属性，请运行以下命令：

   ```sh
   az disk show --resource-group $RGNAME --name $DISKNAME
   ```

#### 任务 3：使用 Azure CLI 配置 托管磁盘

在此任务中，你需要在 Cloud Shell 中使用 Azure CLI 会话来管理 Azure 托管磁盘的配置。 

1. 要将 Azure 托管磁盘的大小增加到 **64 GB**，你需要在 Cloud Shell 中从 Bash 会话中运行以下命令：

   ```sh
   az disk update --resource-group $RGNAME --name $DISKNAME --size-gb 64
   ```

1. 要验证更改是否生效，请运行以下命令：

   ```sh
   az disk show --resource-group $RGNAME --name $DISKNAME --query diskSizeGb
   ```

1. 要将“磁盘性能 SKU”更改为 **“Premium_LRS”**，需要在 Cloud Shell 中从 Bash 会话中运行以下命令：

   ```sh
   az disk update --resource-group $RGNAME --name $DISKNAME --sku 'Premium_LRS'
   ```

1. 要验证更改是否生效，请运行以下命令：

   ```sh
   az disk show --resource-group $RGNAME --name $DISKNAME --query sku
   ```

#### 清理资源

   >**说明**：请记得移除任何你不再使用的新创建的 Azure 资源。移除未使用的资源，确保不产生意外费用。

1. 在 Azure 门户中，在 **“Cloud Shell”** 中启动 **“Bash”** shell 会话。

1. 通过运行以下命令，列出本模块在实验室中创建的所有资源组：

   ```sh
   az group list --query "[?starts_with(name,'az104-03')].name" --output tsv
   ```

1. 通过运行以下命令，删除本模块在实验室中创建的所有资源组：

   ```sh
   az group list --query "[?starts_with(name,'az104-03')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

    >**注意**：该命令异步执行（由 -- nowait 参数确定），因此之后尽管你能够立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能移除资源组。

#### 回顾

在本实验室中，你已：

- 在 Azure Cloud Shell 中启动 Bash 会话
- 使用 Azure CLI 创建一个资源组和 Azure 托管磁盘
- 使用 Azure CLI 配置托管磁盘