---
lab:
    title: '实验室 09c - 实现 Azure Kubernetes 服务'
    module: '模块 09 - 无服务器计算'
---

# 实验室 09c - 实施 Azure Kubernetes 服务
# 学生实验室手册

## 实验场景

Contoso 具有许多不适合使用 Azure 容器实例运行的多层应用程序。为确定它们是否可作为容器化工作负荷运行，希望评估 Kubernetes 作为容器业务流程协调程序的使用情况。为了进一步减少管理开销，你希望测试 Azure Kubernetes 服务，包括该服务的简化部署体验和缩放功能。

## 目标

你将在本实验室中了解以下内容：

+ 任务 1：部署 Azure Kubernetes 服务群集
+ 任务 2：将 Pod 部署到 Azure Kubernetes 服务群集
+ 任务 3：缩放 Azure Kubernetes 服务群集中的容器化工作负载

## 预计用时：40 分钟

## 说明

### 练习 1

#### 任务 1：注册 Microsoft.Kubernetes 和 Microsoft.KubernetesConfiguration 资源提供程序。

在此任务中，将注册部署 Azure Kubernetes 服务群集所需的资源提供程序。

1. 登录至 [Azure 门户](https://portal.azure.com)。

1. 在 Azure 门户中，单击 Azure 门户右上方的图标，打开 **Azure Cloud Shell**。

1. 提示选择 **“Bash”** 或 **“PowerShell”** 时，选择 **“PowerShell”**。 

    >**注意**：如果这是第一次启动 **“Cloud Shell”**，且看到 **“未装载任何存储”** 消息，请选择在本实验室中使用的订阅，然后选择 **“创建存储”**。 

1. 在“Cloud Shell”窗格中运行以下命令，以注册 Microsoft.Insights 和 Microsoft.AlertsManagement 资源提供程序。

   ```pwsh
   Register-AzResourceProvider -ProviderNamespace Microsoft.Kubernetes
   
   Register-AzResourceProvider -ProviderNamespace Microsoft.KubernetesConfiguration
   ```

1. 关闭“Cloud Shell”窗格。   


#### 任务 2：部署 Azure Kubernetes 服务群集

在此任务中，将使用 Azure 门户部署 Azure Kubernetes 服务群集。

1. 在 Azure 门户中，搜索查找 **“Kubernetes 服务”**，然后在 **“Kubernetes 服务”** 边栏选项卡上，选择 **“+ 添加”**。 

1. 在 **“创建 Kubernetes 群集”** 边栏选项卡的 **“基本”** 选项卡中，指定以下设置（将其他设置保留为默认值）：

    | 设置 | 数值 |
    | ---- | ---- |
    | 订阅 | 在本实验室使用的 Azure 订阅的名称 |
    | 资源组 | 新资源组名称 **az104-09c-rg1** |
    | Kubernetes 群集名称 | **az104-9c-aks1** |
    | 区域 | 可在其中预配 Kubernetes 群集的区域名称 |
    | Kubernetes 版本 | 接受默认值 |
    | 节点大小 | 接受默认值 |
    | 节点计数 | **1** |

1. 单击 **“下一步: “节点池 >”**，然后在 **“创建 Kubernetes 群集”** 边栏选项卡的 **“节点池”** 选项卡中，指定以下设置（将其他设置保留为默认值）：

    | 设置 | 数值 |
    | ---- | ---- |
    | 虚拟节点 | **禁用** |
    | VM 规模集 | **已启用** |
	
1. 单击 **“下一步: “身份验证 >”**，然后在 **“创建 Kubernetes 集群”** 边栏选项卡的 **“身份验证”** 选项卡上，指定以下设置（将其他设置保留为默认值）：

    | 设置 | 数值 |
    | ---- | ---- |
    | 服务主体 | 接受默认值 |
    | 启用 RBAC | **是** |


1. 单击“下一步:**联网 >**，然后在 **“创建 Kubernetes 集群”** 边栏选项卡的 **“联网”** 选项卡上，指定以下设置（将其他设置保留为默认值）：

    | 设置 | 数值 |
    | ---- | ---- |
    | 网络配置 | **高级** |
    | DNS 名称前缀 | 任何有效的、全局唯一的 DNS 主机名 |
    | 负载均衡器 | **标准** |
    | HTTP 应用程序路由 | **否** |

1. 单击 **“下一步: 集成 >”**，然后在 **“创建 Kubernetes 群集”** 边栏选项卡的 **“集成”** 选项卡中，将 **“容器监视”** 设置为 **“禁用”**，单击 **“查看 + 创建”**，然后单击 **“创建”**。 

    >**注意**：在生产方案中，你需要启用监视。在这种情况下，由于实验室未涵盖监视，因此禁用监视。 

    >**注意**：等待部署完成。该操作需约 10 分钟。


#### 任务 3：将 Pod 部署到 Azure Kubernetes 服务群集

在此任务中，将 Pod 部署到 Azure Kubernetes 服务群集中。

1. 在“部署”边栏选项卡中，单击 **“前往资源”** 链接。

1. 在 **“az104-9c-aks1”** Kubernetes 服务边栏选项卡的 **“设置”** 部分，单击 **“节点池”**。

1. 在 **“az104-9c-aks1” - “节点池”** 边栏选项卡中，请验证群集是否包含具有一个节点的单个池。

1. 在 Azure 门户中，单击 Azure 门户右上方的图标，打开 **Azure Cloud Shell**。

1. 将 **“Azure Cloud Shell”** 切换到 **“Bash”**（黑色背景）。

1. 在“Cloud Shell”窗格中，运行以下命令以检索用于访问 AKS 群集的凭据：

    ```sh
    RESOURCE_GROUP='az104-09c-rg1'

    AKS_CLUSTER='az104-9c-aks1'

    az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER
    ``` 

1. 在 **“Cloud Shell”** 窗格中运行以下命令，以验证到 AKS 群集的连接：

    ```sh
    kubectl get nodes
    ```

1. 在 **“Cloud Shell”** 窗格中，检查输出并验证群集包含的一个节点此时是否正在报告 **“就绪”** 状态。 

1. 在 **“Cloud Shell”** 窗格中运行以下命令，以从 Docker Hub 部署 **nginx** 映像：

    ```sh
    kubectl create deployment nginx-deployment --image=nginx
    ```

    > **注意**：键入部署名称 (nginx-deployment) 时，请确保使用小写字母

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以验证是否已创建 Kubernetes pod：

    ```sh
    kubectl get pods
    ```

1. 在 **“Cloud Shell”** 窗格中，运行下列命令以确定部署状态：

    ```sh
    kubectl get deployment
    ```

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以使 pod 可从 Internet 访问：

    ```sh
    kubectl expose deployment nginx-deployment --port=80 --type=LoadBalancer
    ```

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以确定是否已预配公共 IP 地址：

    ```sh
    kubectl get service
    ```

1. 重新运行该命令，直到 **“nginx-deployment”** 条目的 **“EXTERNAL-IP”** 列中的值从 **“\<pending\>”** 更改为公共 IP 地址。记录 **“nginx-deployment”** 的 **“EXTERNAL-IP”** 列中的公共 IP 地址。

1. 打开浏览器窗口，然后导航到上一步中获取的 IP 地址。验证浏览器页面是否显示 **“欢迎使用 nginx！”** 消息。

#### 任务 4：缩放 Azure Kubernetes 服务群集中的容器化工作负载

在此任务中，你将水平缩放 Pod 的数量和群集节点的数量。

1. 在 **“Cloud Shell”** 窗格中运行以下命令，以通过将 Pod 数增加到 2 来扩展部署：

    ```sh
    kubectl scale --replicas=2 deployment/nginx-deployment
    ```

1. 在 **“Cloud Shell”** 窗格中运行以下命令，以验证扩展部署的结果：

    ```sh
    kubectl get pods
    ```

    > **注意**：查看命令的输出并验证 pod 的数量是否增加到 2。

1. 在 **“Clud Shell”** 窗格中，运行以下命令以通过将节点数增加到 2 来横向扩展群集：

    ```sh
    az aks scale --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER --node-count 2
    ```

    > **注意**：等待附加节点配置完成。该操作需要约 3 分钟。如果失败，请重新运行 `az aks scale` 命令。

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以验证扩展群集的结果：

    ```sh
    kubectl get nodes
    ```

    > **注意**：查看命令的输出并验证节点数是否增加到 2。

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以扩展部署：

    ```
    kubectl scale --replicas=10 deployment/nginx-deployment
    ```

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以验证部署缩放结果：

    ```
    kubectl get pods
    ```

    > **注意**：查看命令的输出并验证 pod 的数量是否增加到 10。

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以查看群集节点之间的 Pod 分布：

    ```
    kubectl get pod -o=custom-columns=NODE:.spec.nodeName,POD:.metadata.name
    ```

    > **注意**：查看命令的输出并验证 pod 是否分布在两个节点上。

1. 在 **“Cloud Shell”** 窗格中运行以下命令，以删除部署：

    ```
    kubectl delete deployment nginx-deployment
    ```

1. 关闭 **Cloud Shell** 窗格。


#### 清理资源

   >**注意**：切记删除不再使用的所有新创建的 Azure 资源。删除未使用的资源，确保不产生意外费用。

1. 在 Azure 门户中，在 **“Cloud Shell”** 窗格中打开 **“Bash”** shell 会话。

1. 运行以下命令，列出在本模块各实验室中创建的所有资源组：

   ```sh
   az group list --query "[?starts_with(name,'az104-09c')].name" --output tsv
   ```

1. 运行以下命令，删除在本模块各个实验室中创建的所有资源组：

   ```sh
   az group list --query "[?starts_with(name,'az104-09c')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

    >**注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

#### 回顾

在本实验室中，你已：

- 部署 Azure Kubernetes 服务群集
- 将 Pod 部署到 Azure Kubernetes 服务群集
- 缩放 Azure Kubernetes 服务群集中的容器化工作负载
