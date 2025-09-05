---
lab:
  topic: Azure events and messaging
  title: 从 Azure 服务总线发送和接收消息
  description: 了解如何使用 .NET Azure.Messaging.ServiceBus SDK 从 Azure 服务总线发送和接收消息。
---

# 从 Azure 服务总线发送和接收消息

在此练习中，将创建和配置 Azure 服务总线资源，然后使用 Azure.Messaging.ServiceBus SDK 生成一个 .NET 应用以发送和接收消息****。 了解如何预配服务总线命名空间和队列、分配权限以及以编程方式与消息交互。 

在本练习中执行的任务：

* 创建 Azure 服务总线资源
* 将角色分配给 Microsoft Entra 用户名
* 创建 .NET 控制台应用以发送和接收消息
* 清理资源

此练习大约需要 30 分钟才能完成****。

## 创建 Azure 事件中心资源

在练习的此部分，你将使用 Azure CLI 在 Azure 中创建所需的资源。

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***Bash*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。 如果系统提示你选择存储帐户来保存文件，请选择“不需要存储帐户”、你的订阅，然后选择“应用”********。

    > **备注**：如果以前创建了使用 *PowerShell* 环境的 Cloud Shell，请将其切换到 ***Bash***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

1. 为此练习所需的资源创建资源组。 如果已有要使用的资源组，请继续执行下一步。 将 myResourceGroup 替换为你希望在资源组中使用的名称****。 如果需要，可以将 useast 替换为附近的区域****。

    ```
    az group create --name myResourceGroup --location eastus
    ```

1. 许多命令都需要唯一的名称并使用相同的参数。 创建一些变量可以减少对创建资源的命令所需做的修改。 运行以下命令以创建所需的变量。 将 myResourceGroup 替换为你在本次练习中使用的名称****。 如果在上一步中更改了位置，请对 location 变量进行相同的更改****。

    ```
    resourceGroup=myResourceGroup
    location=eastus
    namespaceName=svcbusns$RANDOM
    ```

1. 在本练习的后面部分，你将需要分配给命名空间的名称。 运行以下命令并记录输出。

    ```
    echo $namespaceName
    ```

### 创建 Azure 服务总线命名空间和队列

1. 创建服务总线消息传递命名空间。 以下命令使用之前创建的变量创建一个命名空间。 此操作需要数分钟才能完成。

    ```bash
    az servicebus namespace create \
        --resource-group $resourceGroup \
        --name $namespaceName \
        --location $location
    ```

1. 创建命名空间后，需要创建一个队列来保存消息。 运行以下命令以创建名为 myqueue 的队列****。

    ```bash
    az servicebus queue create --resource-group $resourceGroup \
        --namespace-name $namespaceName \
        --name myqueue
    ```

### 将角色分配给 Microsoft Entra 用户名

为了让你的应用能够发送和接收消息，请将 Microsoft Entra 用户分配到服务总线命名空间级别的“Azure 服务总线数据所有者”角色****。 这将授予用户帐户通过 Azure RBAC 管理和访问队列和主题的权限。 在 Cloud Shell 中执行以下步骤。

1. 运行以下命令，从你的帐户中检索 userPrincipalName****。 这代表该角色将被分配给的对象。

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. 运行以下命令以检索服务总线命名空间的资源 ID。 该资源 ID 会将角色分配的范围限定为特定的命名空间。

    ```
    resourceID=$(az servicebus namespace show --name $namespaceName \
        --resource-group $resourceGroup \
        --query id --output tsv)
    ```
1. 运行以下命令以创建和分配“Azure 服务总线数据所有者”角色****。

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Azure Service Bus Data Owner" \
        --scope $resourceID
    ```

## 创建 .NET 控制台应用以发送和接收消息

在将所需的资源部署到 Azure 后，下一步是设置控制台应用程序。 以下步骤均在 Cloud Shell 中执行。

>**
          **提示：通过拖动上边框调整 Cloud Shell 的大小以显示更多信息和代码。 还可以使用最小化和最大化按钮在 Cloud Shell 和主门户界面之间切换。

1. 运行以下命令创建一个目录来存放项目，并切换到该项目目录中。

    ```
    mkdir svcbus
    cd svcbus
    ```

1. 创建 .NET 控制台应用程序。

    ```
    dotnet new console
    ```

1. 运行以下命令，将 Azure.Messaging.ServiceBus 和 Azure.Identity 包添加到项目********。

    ```
    dotnet add package Azure.Messaging.ServiceBus
    dotnet add package Azure.Identity
    ```

### 添加项目的起始代码

1. 在 Cloud Shell 中运行以下命令，以开始编辑应用程序。

    ```
    code Program.cs
    ```

1. 将任何现有内容替换为以下代码。 请务必查看代码中的注释，并将 <YOUR-NAMESPACE> 替换为你之前记录的服务总线命名空间****。

    ```csharp
    using Azure.Messaging.ServiceBus;
    using Azure.Identity;
    using System.Timers;
    
    
    // TODO: Replace <YOUR-NAMESPACE> with your Service Bus namespace
    string svcbusNameSpace = "<YOUR-NAMESPACE>.servicebus.windows.net";
    string queueName = "myQueue";
    
    
    // ADD CODE TO CREATE A SERVICE BUS CLIENT
    
    
    
    // ADD CODE TO SEND MESSAGES TO THE QUEUE
    
    
    
    // ADD CODE TO PROCESS MESSAGES FROM THE QUEUE
    
    
    
    // Dispose client after use
    await client.DisposeAsync();
    ```

1. 按 Ctrl+S 保存所做的更改****。

### 添加向队列发送消息的代码

现在是时候添加代码，以创建服务总线客户端并向队列发送一批消息了。

1. 找到“// ADD CODE TO CREATE A SERVICE BUS CLIENT”注释，并在该注释后直接添加以下代码****。 请务必查看代码和注释。

    ```csharp
    // Create a DefaultAzureCredentialOptions object to configure the DefaultAzureCredential
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create a Service Bus client using the namespace and DefaultAzureCredential
    // The DefaultAzureCredential will use the Azure CLI credentials, so ensure you are logged in
    ServiceBusClient client = new(svcbusNameSpace, new DefaultAzureCredential(options));
    ```

1. 找到“// ADD CODE TO SEND MESSAGES TO THE QUEUE”注释，并在该注释后直接添加以下代码****。 请务必查看代码和注释。

    ```csharp
    // Create a sender for the specified queue
    ServiceBusSender sender = client.CreateSender(queueName);
    
    // create a batch 
    using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();
    
    // number of messages to be sent to the queue
    const int numOfMessages = 3;
    
    for (int i = 1; i <= numOfMessages; i++)
    {
        // try adding a message to the batch
        if (!messageBatch.TryAddMessage(new ServiceBusMessage($"Message {i}")))
        {
            // if it is too large for the batch
            throw new Exception($"The message {i} is too large to fit in the batch.");
        }
    }
    
    try
    {
        // Use the producer client to send the batch of messages to the Service Bus queue
        await sender.SendMessagesAsync(messageBatch);
        Console.WriteLine($"A batch of {numOfMessages} messages has been published to the queue.");
    }
    finally
    {
        // Calling DisposeAsync on client types is required to ensure that network
        // resources and other unmanaged objects are properly cleaned up.
        await sender.DisposeAsync();
    }
    
    Console.WriteLine("Press any key to continue");
    Console.ReadKey();
    ```

1. 按 Ctrl+S 保存文件，然后继续完成练习****。

### 添加代码以处理队列中的消息

1. 找到“// ADD CODE TO PROCESS MESSAGES FROM THE QUEUE”注释，并在该注释后直接添加以下代码****。 请务必查看代码和注释。

    ```csharp
    // Create a processor that we can use to process the messages in the queue
    ServiceBusProcessor processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions());
    
    // Idle timeout in milliseconds, the idle timer will stop the processor if there are no more 
    // messages in the queue to process
    const int idleTimeoutMs = 3000;
    System.Timers.Timer idleTimer = new(idleTimeoutMs);
    idleTimer.Elapsed += async (s, e) =>
    {
        Console.WriteLine($"No messages received for {idleTimeoutMs / 1000} seconds. Stopping processor...");
        await processor.StopProcessingAsync();
    };
    
    try
    {
        // add handler to process messages
        processor.ProcessMessageAsync += MessageHandler;
    
        // add handler to process any errors
        processor.ProcessErrorAsync += ErrorHandler;
    
        // start processing 
        idleTimer.Start();
        await processor.StartProcessingAsync();
    
        Console.WriteLine($"Processor started. Will stop after {idleTimeoutMs / 1000} seconds of inactivity.");
        // Wait for the processor to stop
        while (processor.IsProcessing)
        {
            await Task.Delay(500);
        }
        idleTimer.Stop();
        Console.WriteLine("Stopped receiving messages");
    }
    finally
    {
        // Dispose processor after use
        await processor.DisposeAsync();
    }
    
    // handle received messages
    async Task MessageHandler(ProcessMessageEventArgs args)
    {
        string body = args.Message.Body.ToString();
        Console.WriteLine($"Received: {body}");
    
        // Reset the idle timer on each message
        idleTimer.Stop();
        idleTimer.Start();
    
        // complete the message. message is deleted from the queue. 
        await args.CompleteMessageAsync(args.Message);
    }
    
    // handle any errors when receiving messages
    Task ErrorHandler(ProcessErrorEventArgs args)
    {
        Console.WriteLine(args.Exception.ToString());
        return Task.CompletedTask;
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

1. 运行以下命令启动控制台应用。 应用会在多个不同阶段暂停，并提示你按任意键继续。 这为你提供了一个在 Azure 门户中查看消息的机会。

    ```
    dotnet run
    ```

    

1. 在 Azure 门户中，导航到所创建的服务总线命名空间。 

1. 选择“概述”窗口底部的 myqueue********。

1. 在左侧导航窗格中，选择“Service Bus Explorer”****。

1. 选择“从开头查看”选项，几秒钟后，这三条消息就应会显示出来****。

1. 在 Cloud Shell 中，按任意键继续，应用程序将处理这三条消息。 
 
1. 应用程序完成消息处理后返回到门户。 再次选择“从开头查看”，会发现队列中已没有消息****。

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。 

