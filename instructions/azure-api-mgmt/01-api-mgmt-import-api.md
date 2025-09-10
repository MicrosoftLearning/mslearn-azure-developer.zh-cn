---
lab:
  topic: Azure API Management
  title: 使用 Azure API Management 导入和配置 API
  description: 了解如何导入、发布和测试符合 OpenAPI 规范的 API。
---

# 使用 Azure API Management 导入和配置 API

在此练习中，将创建 Azure API Management 实例、导入 OpenAPI 规范后端 API、配置 API 设置（包括 Web 服务 URL 和订阅要求）以及测试 API 操作以验证它们是否运行正常。

在本练习中执行的任务：

* 创建 Azure API 管理 (APIM) 实例
* 导入 API
* 配置后端设置
* 测试 API

此练习大约需要 20 分钟才能完成****。

## 创建 API 管理实例

在本练习部分，你将创建资源组和 Azure 存储帐户。 还会记录帐户的终结点和访问密钥。

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***Bash*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。 如果系统提示你选择存储帐户来保存文件，请选择“不需要存储帐户”、你的订阅，然后选择“应用”********。

    > **备注**：如果以前创建了使用 *PowerShell* 环境的 Cloud Shell，请将其切换到 ***Bash***。

1. 为此练习所需的资源创建资源组。 将 myResourceGroup 替换为你希望在资源组中使用的名称****。 如果需要，可以将 eastus2 替换为附近的区域****。 如果已有要使用的资源组，请继续执行下一步。

    ```azurecli
    az group create --location eastus2 --name myResourceGroup
    ```

1. 创建几个供 CLI 命令使用的变量，这样可以减少输入量。 将 myLocation 替换为你之前选择的值****。 APIM 名称必须是全局唯一的名称，且以下脚本将生成一个随机字符串。 将 myEmail 替换为你可以访问的电子邮件地址****。

    ```bash
    myApiName=import-apim-$RANDOM
    myLocation=myLocation
    myEmail=myEmail
    ```

1. 创建 APIM 实例。 az apim create 命令用于创建实例****。 将 myResourceGroup 替换为你之前选择的值****。

    ```bash
    az apim create -n $myApiName \
        --location $myLocation \
        --publisher-email $myEmail  \
        --resource-group myResourceGroup \
        --publisher-name Import-API-Exercise \
        --sku-name Consumption 
    ```
    > **注意：** 操作应在大约五分钟内完成。 

## 导入后端 API

本部分演示如何导入和发布 OpenAPI 规范后端 API。

1. 在 Azure 门户中搜索并选择“API 管理服务” 。

1. 在“API 管理服务”屏幕上，选择你创建的 API 管理实例****。

1. 在“API 管理服务”导航窗格中，选择“**”>“API”，然后选择“API”**********。

    ![导航窗格中 API 部分的屏幕截图。](./media/select-apis-navigation-pane.png)


1. 在“从定义创建”部分，选择 OpenAPI，并在显示的弹出窗口中将“基本/完全”切换设置为“完全”****************。

    ![OpenAPI 对话框的屏幕截图。 下表详细介绍了这些字段。](./media/create-api.png)

    使用下表中的值填写表单。 你可以将任何未提及的字段保留为其默认值。

    | 设置 | 值 | 说明 |
    |--|--|--|
    | **OpenAPI 规范** | `https://bigconference.azurewebsites.net/` | 引用实现 API 的服务，将请求转发到此地址。 输入此值后，表单中的大部分必填信息都会自动填充。 |
    | **URL 方案** | 选择 **HTTPS**。 | 定义 API 接受的 HTTP 协议的安全级别。 |

1. 选择**创建**。

## 配置 API 设置

此时将创建“Big Conference API”。** 接下来配置 API 设置。 

1. 在菜单中选择“设置”****。

1. 在“Web 服务 URL”字段中输入 `https://bigconference.azurewebsites.net/`****。

1. 取消选择“需要订阅”复选框。

1. 选择“保存”。

## 测试 API

现在已导入并配置了 API，可以测试 API 了。

1. 在菜单栏中选择“测试”****。 这将显示 API 中可用的所有操作。

1. 搜索并选择 Speakers_Get 操作****。 

1. 选择**Send**。 可能需要向下滚动页面才能查看 HTTP 响应。

    后端以“200 正常”和某些数据做出响应 。

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。
1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。
