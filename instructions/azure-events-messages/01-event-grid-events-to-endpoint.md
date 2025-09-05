---
lab:
  topic: Azure events and messaging
  title: 使用 Azure 事件网格将事件路由到自定义终结点
  description: 了解如何使用 Azure 事件网格将事件路由到自定义终结点。
---

# 使用 Azure 事件网格将事件路由到自定义终结点

在此练习中，你将创建 Azure 事件网格主题和 Web 应用终结点，然后生成 .NET 控制台应用程序，将自定义事件发送到事件网格主题。 了解如何配置事件订阅、使用事件网格进行身份验证，以及通过在 Web 应用中查看事件来验证事件是否已成功路由到终结点。

在本练习中执行的任务：

* 创建 Azure 事件网格资源
* 启用事件网格资源提供程序
* 在事件网格中创建主题
* 创建消息终结点
* 订阅主题
* 使用 .NET 控制台应用发送事件
* 清理资源

此练习大约需要 30 分钟才能完成****。

## 创建 Azure 事件网格资源

在练习的此部分，你将使用 Azure CLI 在 Azure 中创建所需的资源。

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***Bash*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。 如果系统提示你选择存储帐户来保存文件，请选择“不需要存储帐户”、你的订阅，然后选择“应用”********。

    > **备注**：如果以前创建了使用 *PowerShell* 环境的 Cloud Shell，请将其切换到 ***Bash***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

1. 为此练习所需的资源创建资源组。 如果已有要使用的资源组，请继续执行下一步。 将 myResourceGroup 替换为你希望在资源组中使用的名称****。 如果需要，可以将 useast 替换为附近的区域****。

    ```bash
    az group create --name myResourceGroup --location eastus
    ```

1. 许多命令都需要唯一的名称并使用相同的参数。 创建一些变量可以减少对创建资源的命令所需做的修改。 运行以下命令以创建所需的变量。 将 myResourceGroup 替换为你在本次练习中使用的名称****。 如果在上一步中更改了位置，请对 location 变量进行相同的更改****。

    ```bash
    let rNum=$RANDOM
    resourceGroup=myResourceGroup
    location=eastus
    topicName="mytopic-evgtopic-${rNum}"
    siteName="evgsite-${rNum}"
    siteURL="https://${siteName}.azurewebsites.net"
    ```

### 启用事件网格资源提供程序

Azure 资源提供程序是一项服务，用于定义和管理 Azure 中特定类型的资源。 这是你部署或管理资源时，Azure 在后台所使用的技术。 使用 az provider register 命令注册事件网格资源提供程序****。 

```bash
az provider register --namespace Microsoft.EventGrid
```

注册可能需要几分钟的时间才能完成。 可使用以下命令检查状态。

```bash
az provider show --namespace Microsoft.EventGrid --query "registrationState"
```

> **注意：** 只有以前未使用过事件网格的订阅才需要执行此步骤。

### 在事件网格中创建主题

使用 az eventgrid topic create 命令创建一个主题****。 名称必须是唯一的，因为它是 DNS 条目的一部分。  

```bash
az eventgrid topic create --name $topicName \
    --location $location \
    --resource-group $resourceGroup
```

### 创建消息终结点

在订阅自定义主题之前，我们需要为事件消息创建终结点。 通常情况下，终结点基于事件数据执行操作。 以下脚本使用一个预生成的 Web 应用来显示事件消息。 所部署的解决方案包括应用服务计划、应用服务 Web 应用和 GitHub 中的源代码。

1. 运行以下命令以创建消息终结点。 echo 命令将显示终结点的网站 URL****。

    ```bash
    az deployment group create \
        --resource-group $resourceGroup \
        --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/main/azuredeploy.json" \
        --parameters siteName=$siteName hostingPlanName=viewerhost
    
    echo "Your web app URL: ${siteURL}"
    ```

    > **注意：** 此命令可能需要花费几分钟时间完成。

1. 在浏览器中打开一个新选项卡，导航到上述脚本末尾处生成的 URL，以确保 Web 应用正在运行。 应会看到站点上当前未显示任何消息。

    > **
          **提示：保持浏览器运行状态，它用于显示更新。

### 订阅主题

订阅事件网格主题，以告知事件网格要跟踪哪些事件，以及要将这些事件发送到何处。 

1. 使用 az eventgrid event-subscription create 命令订阅一个主题****。 以下脚本会从你的帐户中检索订阅 ID，并用于创建事件订阅。

    ```bash
    endpoint="${siteURL}/api/updates"
    topicId=$(az eventgrid topic show --resource-group $resourceGroup \
        --name $topicName --query "id" --output tsv)
    
    az eventgrid event-subscription create \
        --source-resource-id $topicId \
        --name TopicSubscription \
        --endpoint $endpoint
    ```

1. 再次查看 Web 应用，并注意现已向该应用发送了订阅验证事件。 选择眼睛图标以展开事件数据。 事件网格发送验证事件，以便终结点可以验证它是否想要接收事件数据。 Web 应用包含用于验证订阅的代码。

## 使用 .NET 控制台应用程序发送事件

在将所需的资源部署到 Azure 后，下一步是设置控制台应用程序。 以下步骤均在 Cloud Shell 中执行。

>**
          **提示：通过拖动上边框调整 Cloud Shell 的大小以显示更多信息和代码。 还可以使用最小化和最大化按钮在 Cloud Shell 和主门户界面之间切换。

1. 运行以下命令创建一个目录来存放项目，并切换到该项目目录中。

    ```bash
    mkdir eventgrid
    cd eventgrid
    ```

1. 创建 .NET 控制台应用程序。

    ```bash
    dotnet new console
    ```

1. 运行以下命令，将 Azure.Messaging.EventGrid 和 dotenv.net 包添加到项目********。

    ```bash
    dotnet add package Azure.Messaging.EventGrid
    dotnet add package dotenv.net
    ```

### 配置控制台应用程序

在本部分中，你将检索主题终结点和访问密钥，以便将其添加到 .env 文件以保存这些机密****。

1. 运行以下命令，检索你之前创建的主题的 URL 和访问密钥。 请务必记录这些值。

    ```bash
    az eventgrid topic show --name $topicName -g $resourceGroup --query "endpoint" --output tsv
    az eventgrid topic key list --name $topicName -g $resourceGroup --query "key1" --output tsv
    ```

1. 运行以下命令以创建 .env 文件来保存机密，然后在代码编辑器中打开该文件****。

    ```bash
    touch .env
    code .env
    ```

1. 将以下代码添加到 .env 文件****。 将 YOUR_TOPIC_ENDPOINT 和 YOUR_TOPIC_ACCESS_KEY 替换为你之前记录的值********。

    ```
    TOPIC_ENDPOINT="YOUR_TOPIC_ENDPOINT"
    TOPIC_ACCESS_KEY="YOUR_TOPIC_ACCESS_KEY"
    ```

1. 按 Ctrl+S 保存文件，然后按 Ctrl+Q 退出编辑器********。

现在，可以使用 Cloud Shell 中的编辑器替换 **Program.cs** 文件中的模板代码。

### 添加项目所需的代码

1. 在 Cloud Shell 中运行以下命令，以开始编辑应用程序。

    ```bash
    code Program.cs
    ```

1. 将任何现有代码替换为以下代码代码。 请务必查看代码中的注释。

    ```csharp
    using dotenv.net; 
    using Azure.Messaging.EventGrid; 
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    
    // Start the asynchronous process to send an Event Grid event
    ProcessAsync().GetAwaiter().GetResult();
    
    async Task ProcessAsync()
    {
        // Retrieve Event Grid topic endpoint and access key from environment variables
        var topicEndpoint = envVars["TOPIC_ENDPOINT"];
        var topicKey = envVars["TOPIC_ACCESS_KEY"];
        
        // Check if the required environment variables are set
        if (string.IsNullOrEmpty(topicEndpoint) || string.IsNullOrEmpty(topicKey))
        {
            Console.WriteLine("Please set TOPIC_ENDPOINT and TOPIC_ACCESS_KEY in your .env file.");
            return;
        }
    
        // Create an EventGridPublisherClient to send events to the specified topic
        EventGridPublisherClient client = new EventGridPublisherClient
            (new Uri(topicEndpoint),
            new Azure.AzureKeyCredential(topicKey));
    
        // Create a new EventGridEvent with sample data
        var eventGridEvent = new EventGridEvent(
            subject: "ExampleSubject",
            eventType: "ExampleEventType",
            dataVersion: "1.0",
            data: new { Message = "Hello, Event Grid!" }
        );
    
        // Send the event to Azure Event Grid
        await client.SendEventAsync(eventGridEvent);
        Console.WriteLine("Event sent successfully.");
    }
    ```

1. 按 Ctrl+S 保存文件，然后按 Ctrl+Q 退出编辑器********。

## 登录到 Azure 并运行应用

1. 在 Cloud Shell 命令行窗格中，输入以下命令以登录到 Azure。

    ```
    az login
    ```

    **<font color="red">必须登录到 Azure - 即使 Cloud Shell 会话已经过身份验证。</font>**

    > **备注**：在大多数情况下，仅使用 *az login* 就足够了。 但是，如果在多个租户中有订阅，则可能需要使用 *--tenant* 参数指定租户。 有关详细信息，请参阅[使用 Azure CLI 以交互方式登录到 Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)。

1. 在 Cloud Shell 中运行以下命令，启动控制台应用程序。 当消息发送成功后，你会看到“已成功发送事件。”**** 消息。

    ```bash
    dotnet run
    ```

1. 查看 Web 应用以查看刚刚发送的事件。 选择眼睛图标以展开事件数据。

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。