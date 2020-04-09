---
lab:
    title: '09c - 实现 Azure Kubernetes 服务'
    module: '模块 09 - 无服务器计算'
---

# 实验室 09c - 实现 Azure Kubernetes 服务
# 学生实验室手册

## 实验室场景

Contoso 有许多不适合使用 Azure 容器实例运行的多层应用程序。为了确定它们是否可以作为容器化工作负荷来运行，你想评估使用 Kubernetes 作为容器业务流程协调程序。为了进一步减少管理开销，你想测试 Azure Kubernetes 服务，包括其简化的部署体验和缩放功能。

## 目标

在本实验室中，你将：

+ 任务 1：部署一个 Azure Kubernetes 服务群集
+ 任务 2：将 Pod 部署到 Azure Kubernetes 服务群集
+ 任务 3：缩放 Azure Kubernetes 服务群集中的容器化工作负荷

## 说明

### 练习 1：

#### 任务 1：部署一个 Azure Kubernetes 服务群集

在此任务中，你将使用 Azure 门户部署 Azure Kubernetes 服务群集。

1. 登录到 [Azure 门户](https://portal.azure.com)。

1. 在 Azure 门户中，搜索定位 **“Kubernetes 服务”** 然后，在 **“Kubernetes 服务”** 边栏选项卡中，单击 **“+ 添加”**。 

1. 在 **“创建 Kubernetes 集群”** 边栏选项卡的 **“基本”** 选项卡，指定以下设置（保留其他设置的默认值）：

    | 设置 | 数值 |
    | ---- | ---- |
    | 订阅 | 你在本实验室中使用的 Azure 订阅的名称 |
    | 资源组 | 一个新资源组的名称**az104-09c-rg1** |
    | Kubernetes 群集的名称 | **az104-9c-aks1** |
    | 区域 | 你可以在其中预配一个 Kubernetes 群集的区域的名称 |
    | Kubernetes 版本 | 接受默认 |
    | DNS 名称前缀 | 任何有效的、全局唯一的 DNS 主机名 |
    | 节点大小 | 接受默认 |
    | 节点数量 | **1** |

1. 点击 **“下一步： 规模 >”**，并在 **“创建 Kubernetes 群集”** 边栏选项卡的 **“规模”** 选项卡指定以下设置（保留其他设置的默认值）：

    | 设置 | 数值 |
    | ---- | ---- |
    | 虚拟节点 | **禁用** |
    | VM 规模集 | **已启用** |
	
1. 点击 **“下一步： 身份验证 >”**，同时，在 **“创建 Kubernetes 群集”** 边栏选项卡中的 **“身份认证”** 选项卡上 ，指定以下设置（其他设置则保留默认值）：

    | 设置 | 数值 |
    | ---- | ---- |
    | 服务主体 | 接受默认 |
    | 启用 RBAC | **是** |


1. 点击 **“下一步： 网络>”** ，同时，在 **“创建 Kubernetes 群集”** 边栏选项卡中的 **“网络”** 选项卡上，指定以下设置（其他设置则保留默认值）：

    | 设置 | 数值 |
    | ---- | ---- |
    | HTTP 应用程序路由 | **否** |
    | 负载均衡器 | **标准** |
    | 网络配置 | **高级** |

1. 点击 **“下一步： 监视>”**，在 **“创建 Kubernetes 集群”** 边栏选项卡中的 **“监视”** 选项卡，把 **“启用容器监视”** 设置为 **“No”**，单击 **“查看 + 创建”**，然后单击 **“创建”**。 

    >**说明**：在生产方案中，需要启用监视。在这种情况下，监控是禁用的，因为它不在实验范围内。 

    >**说明**：等待部署完成。该操作需约 10 分钟。


#### 任务 2：将 Pod 部署到 Azure Kubernetes 服务群集

在此任务中，你将在 Azure Kubernetes 服务集群中部署一个 pod。

1. 在“部署”边栏选项卡中，单击 **“前往资源”** 链接。

1. 在 **“az104-9c-aks1”** Kubernetes 服务边栏选项卡中的 **“设置”** 部分，单击 **“节点池”**。

1. 在 **“az104-9c-aks1 - 节点池”** 边栏选项卡中，验证群集是否由具有一个节点的单个池组成。

1. 在 Azure 门户中，单击 Azure 门户右上角的图标打开 **“Azure Cloud Shell”**。

1. 如果提示你选择 **“Bash”** 或者 **“PowerShell”**，选择 **“Bash”**。 

    >**说明**：如果这是你第一次启动 **“Cloud Shell”**，并看到 **“你没有装载存储”** 的消息，请选择你在本实验中使用的订阅，然后单击 **“创建存储”**。 

1. 在“Cloud Shell” 窗格中，运行以下命令以检索访问 AKS 群集所需的凭据：

    ```sh
    RESOURCE_GROUP='az104-09c-rg1'

    AKS_CLUSTER='az104-9c-aks1'

    az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER
    ``` 

1. 在 **“Cloud Shell”** 窗格中，运行下列命令以验证与 AKS 群集的连接性：

    ```sh
    kubectl get nodes
    ```

1. 在 **“Cloud Shell”** 窗格，查看输出，并验证组成群集的一个节点此时是否正在报告 **“准备”** 状态。 

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以从 Docker Hub 中部署 **“nginx”** 映像：

    ```sh
    kubectl create deployment nginx-deployment --image=nginx
    ```

    > **注意**：键入部署名称时（nginx - 部署），确保使用小写字母。

1. 在 **“Cloud Shell”** 窗格中，运行以下命令，以验证是否已创建 Kubernetes pod：

    ```sh
    kubectl get pods
    ```

1. 在 **“Cloud Shell”** 窗格中，运行以下命令，以识别部署状态：

    ```sh
    kubectl get deployment
    ```

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以便可以通过 Internet 使用 pod：

    ```sh
    kubectl expose deployment nginx-deployment --port=80 --type=LoadBalancer
    ```

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以确定是否已预配公共 IP 地址：

    ```sh
    kubectl get service
    ```

1. 重新运行命令，直到 **EXTERNAL-IP** 列的 **nginx-deployment** 条目的值从 **\<待处理\>** 变更为公共 IP 地址。注意 **EXTERNAL-IP** 列的 **nginx-deployment** 的公共 IP 地址。

1. 打开浏览器窗口，然后导航到在上一步获取的 IP 地址。验证浏览器页面是否显示 **Welcome to nginx!** 消息。

#### 任务 3：缩放 Azure Kubernetes 服务群集中的容器化工作负荷

在此任务中，你将水平缩放 Pod 数量，然后是群集节点的数量。

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以通过将 Pod 的数量增加到 2 来缩放部署：

    ```sh
    kubectl scale --replicas=2 deployment/nginx-deployment
    ```

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以验证扩展部署的结果：

    ```sh
    kubectl get pods
    ```

    > **注意**：查看命令的输出并验证 Pod 的数量是否增加到 2。

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以通过将节点数增加到 2 来横向扩展群集：

    ```sh
    az aks scale --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER --node-count 2
    ```

    > **注意**：等待附加节点预配完成。该操作约需 3 分钟。如果失败，请重新运行 `az aks scale` 命令。

1. 在 **“Cloud Shell”** 窗格中，运行下列命令以验证扩展群集的结果：

    ```sh
    kubectl get nodes
    ```

    > **注意**：查看命令的输出并验证节点数是否增加到 2。

1. 在 **“Cloud Shell”** 窗格中，运行下列命令以对部署进行缩放：

    ```
    kubectl scale --replicas=10 deployment/nginx-deployment
    ```

1. 在 **“Cloud Shell”** 窗格中，运行以下命令以验证扩展部署的结果：

    ```
    kubectl get pods
    ```

    > **注意**：查看命令的输出并验证 Pod 的数量是否增加到 10。

1. 在 **“Cloud Shell”** 窗格中，运行下列命令以查看群集节点之间的 Pod 分布：

    ```
    kubectl get pod -o=custom-columns=NODE:.spec.nodeName,POD:.metadata.name
    ```

    > **注意**：查看命令的输出并验证 Pod 是否分布在两个节点上。

1. 在 **“Cloud Shell”** 窗格中，运行下列命令以删除部署：

    ```
    kubectl delete deployment nginx-deployment
    ```

1. 关闭 **“Cloud Shell”** 窗格。


#### 清理资源

   >**说明**：请记得移除任何你不再使用的新创建的 Azure 资源。移除未使用的资源，确保不产生意外费用。

1. 在 Azure 门户中，打开 **“Cloud Shell”** 窗格中的 **“Bash”** shell 会话。

1. 通过运行以下命令，列出本模块在实验室中创建的所有资源组：

   ```sh
   az group list --query "[?starts_with(name,'az104-09c')].name" --output tsv
   ```

1. 通过运行以下命令，删除本模块在实验室中创建的所有资源组：

   ```sh
   az group list --query "[?starts_with(name,'az104-09c')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

    >**备注**：该命令异步执行（由 -- nowait 参数决定），因此，尽管你随后能够在同一 Bash 会话中立即运行另一个 Azure CLI 命令，但要花几分钟时间才能删除资源组。

#### 回顾

在本实验室中，你已：

- 部署 Azure Kubernetes 服务群集
- 将 Pod 部署到 Azure Kubernetes 服务群集中
- 缩放 Azure Kubernetes 服务群集中的容器化工作负荷