---
lab:
  topic: Azure container services
  title: 使用 Azure CLI 命令将容器部署到 Azure 容器实例
  description: 了解如何使用 Azure CLI 命令将容器部署到 Azure 容器实例。
---

# 使用 Azure CLI 命令将容器部署到 Azure 容器实例

在此练习中，将使用 Azure CLI 在 Azure 容器实例 (ACI) 中部署和运行容器。 你将了解如何创建容器组、指定容器设置以及验证容器化应用程序是否在云中正常运行。

在本练习中执行的任务：

* 在 Azure 中创建 Azure 容器实例资源
* 创建并部署容器
* 验证容器是否在运行。

此练习大约需要 15 分钟才能完成****。

## 创建资源组

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***Bash*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。 如果系统提示你选择存储帐户来保存文件，请选择“不需要存储帐户”、你的订阅，然后选择“应用”********。

    > **备注**：如果以前创建了使用 *PowerShell* 环境的 Cloud Shell，请将其切换到 ***Bash***。

1. 为此练习所需的资源创建资源组。 将 myResourceGroup 替换为你希望在资源组中使用的名称****。 如果需要，可以将 useast 替换为附近的区域****。 如果已有要使用的资源组，请继续执行下一步。

    ```
    az group create --location eastus --name myResourceGroup
    ```

## 创建并部署容器

通过在 **az container create** 命令中提供名称、Docker 映像和 Azure 资源组来创建容器。 通过指定 DNS 名称标签将容器公开到 Internet。

1. 运行以下命令，创建用于向 Internet 公开容器的 DNS 名称。 你的 DNS 名称必须是唯一的，请通过 Cloud Shell 运行此命令，以创建包含唯一名称的变量。

    ```bash
    DNS_NAME_LABEL=aci-example-$RANDOM
    ```

1. 运行以下命令以创建一个容器实例。 将 myResourceGroup 和 myLocation 替换为你之前使用过的值********。 操作需要几分钟时间才能完成。

    ```bash
    az container create --resource-group myResourceGroup \
        --name mycontainer \
        --image mcr.microsoft.com/azuredocs/aci-helloworld \
        --ports 80 \
        --dns-name-label $DNS_NAME_LABEL --location myLocation \
        --os-type Linux \
        --cpu 1 \
        --memory 1.5 
    ```

    在上面的命令中，$DNS_NAME_LABEL 用于指定你的 DNS 名称****。 映像名称 mcr.microsoft.com/azuredocs/aci-helloworld 是指运行基本 Node.js Web 应用程序的 Docker 映像****。

完成 az container create 命令后，转到下一部分****。

## 验证容器是否在运行。

可以使用 az container show 命令检查容器生成状态****。 

1. 运行以下命令，检查创建的容器的预配状态。 将 myResourceGroup 替换为你之前使用过的值****。

    ```bash
    az container show --resource-group myResourceGroup \
        --name mycontainer \
        --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
        --out table 
    ```

    可以看到容器的完全限定的域名 (FQDN) 及其预配状态。 下面是一个示例。

    ```
    FQDN                                    ProvisioningState
    --------------------------------------  -------------------
    aci-wt.eastus.azurecontainer.io         Succeeded
    ```

    > **注意：** 如果容器处于“正在创建”状态，请稍等片刻后再次运行该命令，直至看到“已成功”状态********。

1. 在浏览器中，导航到容器的 FQDN 以查看其正在运行。 你可能会收到一条警告消息，指出网站不安全。

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。
