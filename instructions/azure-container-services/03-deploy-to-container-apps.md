---
lab:
  topic: Azure container services
  title: 使用 Azure CLI 将容器部署到 Azure 容器应用
  description: 了解如何使用 Azure CLI 命令创建安全的 Azure 容器应用环境并部署容器。
---

# 使用 Azure CLI 将容器部署到 Azure 容器应用

在此练习中，将使用 Azure CLI 将容器化应用程序部署到 Azure 容器应用。 你将了解如何创建容器应用环境、部署容器，以及如何验证应用程序是否在 Azure 中正常运行。

在本练习中执行的任务：

* 在 Azure 中创建资源
* 创建 Azure 容器应用环境
* 在环境中部署容器应用

此练习大约需要 15 分钟才能完成****。

## 创建资源组并准备 Azure 环境

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***Bash*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。 如果系统提示你选择存储帐户来保存文件，请选择“不需要存储帐户”、你的订阅，然后选择“应用”********。

    > **备注**：如果以前创建了使用 *PowerShell* 环境的 Cloud Shell，请将其切换到 ***Bash***。

1. 为此练习所需的资源创建资源组。 将 myResourceGroup 替换为你希望在资源组中使用的名称****。 如果需要，可以将 useast 替换为附近的区域****。 如果已有要使用的资源组，请继续执行下一步。

    ```azurecli
    az group create --location eastus --name myResourceGroup
    ```

1. 运行以下命令以确保你安装了最新版本的 Azure CLI 的 Azure 容器应用扩展。

    ```azurecli
    az extension add --name containerapp --upgrade
    ```

### 注册命名空间

需要为 Azure 容器应用注册两个命名空间，请确保按照以下步骤进行注册。 你的订阅中尚未配置这些注册，那么每一项注册都可能需要几分钟时间才能完成。 

1. 注册 Microsoft.AAD 命名空间****。 

    ```bash
    az provider register --namespace Microsoft.App
    ```

1. 为 Azure Monitor Log Analytics 工作区注册 Microsoft.OperationalInsights 提供程序（如果以前未使用过）****。

    ```bash
    az provider register --namespace Microsoft.OperationalInsights
    ```

## 创建 Azure 容器应用环境

Azure 容器应用中的环境围绕一组容器应用创建安全边界。 部署到相同环境的容器应用部署在同一虚拟网络中，并将日志写入同一个 Log Analytics 工作区。

1. 使用 az containerapp env create 命令创建环境****。 将 myResourceGroup 和 myLocation 替换为你之前使用过的值********。 操作需要几分钟时间才能完成。

    ```bash
    az containerapp env create \
        --name my-container-env \
        --resource-group myResourceGroup \
        --location myLocation
    ```

## 在环境中部署容器应用

容器应用环境部署完成后，可以将容器映像部署到你的环境中。

1. 使用 containerapp create 命令部署示例应用容器映像****。 将 myResourceGroup 替换为你之前使用过的值****。

    ```bash
    az containerapp create \
        --name my-container-app \
        --resource-group myResourceGroup \
        --environment my-container-env \
        --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
        --target-port 80 \
        --ingress 'external' \
        --query properties.configuration.ingress.fqdn
    ```

    通过将“--ingress”设置为“external”，可使容器应用能够接收公共请求********。 该命令返回用于访问应用的链接。

    ```
    Container app created. Access your app at <url>
    ```

要验证部署，请选择 az containerapp create 命令返回的 URL，以验证容器应用是否正在运行****。

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。
1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。
