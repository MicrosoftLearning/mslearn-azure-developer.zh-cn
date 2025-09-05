---
lab:
  topic: Azure events and messaging
  title: 从 Azure 队列存储发送和接收消息
  description: 了解如何使用 .NET Azure.StorageQueues SDK 从 Azure 队列存储发送和接收消息。
---

# 从 Azure 队列存储发送和接收消息

在此练习中，将创建和配置 Azure 队列存储资源，然后使用 Azure.Storage.Queues SDK 生成一个 .NET 应用以发送和接收消息****。 你将了解如何预配存储资源、管理队列消息，以及在完成后清理你的环境。 

在本练习中执行的任务：

* 创建 Azure 队列存储资源
* 将角色分配给 Microsoft Entra 用户名
* 创建 .NET 控制台应用以发送和接收消息
* 清理资源

此练习大约需要 30 分钟才能完成****。

## 创建 Azure 队列存储资源

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
    storAcctName=storactname$RANDOM
    ```

1. 在本练习的后面部分，你将需要分配给存储帐户的名称。 运行以下命令并记录输出。

    ```
    echo $storAcctName
    ```

1. 运行以下命令，使用你之前创建的变量来创建存储帐户。 此操作需要数分钟才能完成。

    ```bash
    az storage account create --resource-group $resourceGroup \
        --name $storAcctName --location $location --sku Standard_LRS
    ```

### 将角色分配给 Microsoft Entra 用户名

为了让你的应用能够发送和接收消息，请将你的 Microsoft Entra 用户分配到“存储队列数据参与者”角色****。 这将授予用户帐户使用 Azure RBAC 创建队列和发送/接收消息的权限。 在 Cloud Shell 中执行以下步骤。

1. 运行以下命令，从你的帐户中检索 userPrincipalName****。 这代表该角色将被分配给的对象。

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. 运行以下命令以检索存储帐户的资源 ID。 该资源 ID 会将角色分配的范围限定为特定的命名空间。

    ```
    resourceID=$(az storage account show --resource-group $resourceGroup \
        --name $storAcctName --query id --output tsv)
    ```

1. 运行以下命令以创建和分配“存储队列数据参与者”角色****。

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Storage Queue Data Contributor" \
        --scope $resourceID
    ```

## 创建 .NET 控制台应用以发送和接收消息

在将所需的资源部署到 Azure 后，下一步是设置控制台应用程序。 以下步骤均在 Cloud Shell 中执行。

>**
          **提示：通过拖动上边框调整 Cloud Shell 的大小以显示更多信息和代码。 还可以使用最小化和最大化按钮在 Cloud Shell 和主门户界面之间切换。

1. 运行以下命令创建一个目录来存放项目，并切换到该项目目录中。

    ```
    mkdir queuestor
    cd queuestor
    ```

1. 创建 .NET 控制台应用程序。

    ```
    dotnet new console
    ```

1. 运行以下命令，将 Azure.Storage.Queues 和 Azure.Identity 包添加到项目********。

    ```
    dotnet add package Azure.Storage.Queues
    dotnet add package Azure.Identity
    ```

### 添加项目的起始代码

1. 在 Cloud Shell 中运行以下命令，以开始编辑应用程序。

    ```
    code Program.cs
    ```

1. 将任何现有内容替换为以下代码。 请务必仔细查看代码中的注释，并将 <YOUR-STORAGE-ACCT-NAME> 替换为你之前记录的存储帐户名称****。

    ```csharp
    using Azure;
    using Azure.Identity;
    using Azure.Storage.Queues;
    using Azure.Storage.Queues.Models;
    using System;
    using System.Threading.Tasks;
    
    // Create a unique name for the queue
    // TODO: Replace the <YOUR-STORAGE-ACCT-NAME> placeholder 
    string queueName = "myqueue-" + Guid.NewGuid().ToString();
    string storageAccountName = "<YOUR-STORAGE-ACCT-NAME>";
    
    // ADD CODE TO CREATE A QUEUE CLIENT AND CREATE A QUEUE
    
    
    
    // ADD CODE TO SEND AND LIST MESSAGES
    
    
    
    // ADD CODE TO UPDATE A MESSAGE AND LIST MESSAGES
    
    
    
    // ADD CODE TO DELETE MESSAGES AND THE QUEUE
    
    
    ```

1. 按 Ctrl+S 保存所做的更改****。

### 添加代码以创建队列客户端并创建队列

现在，可以添加代码来创建队列存储客户端并创建队列。

1. 找到“// ADD CODE TO CREATE A QUEUE CLIENT AND CREATE A QUEUE”注释，并在该注释后直接添加以下代码****。 请务必查看代码和注释。

    ```csharp
    // Create a DefaultAzureCredentialOptions object to exclude certain credentials
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Instantiate a QueueClient to create and interact with the queue
    QueueClient queueClient = new QueueClient(
        new Uri($"https://{storageAccountName}.queue.core.windows.net/{queueName}"),
        new DefaultAzureCredential(options));
    
    Console.WriteLine($"Creating queue: {queueName}");
    
    // Create the queue
    await queueClient.CreateAsync();
    
    Console.WriteLine("Queue created, press Enter to add messages to the queue...");
    Console.ReadLine();
    ```

1. 按 Ctrl+S 保存文件，然后继续完成练习****。

### 添加代码以发送消息并列出队列中的消息

1. 找到“// ADD CODE TO SEND AND LIST MESSAGES”注释，并在该注释后直接添加以下代码****。 请务必查看代码和注释。

    ```csharp
    // Send several messages to the queue with the SendMessageAsync method.
    await queueClient.SendMessageAsync("Message 1");
    await queueClient.SendMessageAsync("Message 2");
    
    // Send a message and save the receipt for later use
    SendReceipt receipt = await queueClient.SendMessageAsync("Message 3");
    
    Console.WriteLine("Messages added to the queue. Press Enter to peek at the messages...");
    Console.ReadLine();
    
    // Peeking messages lets you view the messages without removing them from the queue.
    
    foreach (var message in (await queueClient.PeekMessagesAsync(maxMessages: 10)).Value)
    {
        Console.WriteLine($"Message: {message.MessageText}");
    }
    
    Console.WriteLine("\nPress Enter to update a message in the queue...");
    Console.ReadLine();
    ```

1. 按 Ctrl+S 保存文件，然后继续完成练习****。

### 添加代码以更新消息并列出结果

1. 找到“// ADD CODE TO UPDATE A MESSAGE AND LIST MESSAGES”注释，并在该注释后直接添加以下代码****。 请务必查看代码和注释。

    ```csharp
    // Update a message with the UpdateMessageAsync method and the saved receipt
    await queueClient.UpdateMessageAsync(receipt.MessageId, receipt.PopReceipt, "Message 3 has been updated");
    
    Console.WriteLine("Message three updated. Press Enter to peek at the messages again...");
    Console.ReadLine();
    
    
    // Peek messages from the queue to compare updated content
    foreach (var message in (await queueClient.PeekMessagesAsync(maxMessages: 10)).Value)
    {
        Console.WriteLine($"Message: {message.MessageText}");
    }
    
    Console.WriteLine("\nPress Enter to delete messages from the queue...");
    Console.ReadLine();
    ```

1. 按 Ctrl+S 保存文件，然后继续完成练习****。

### 添加代码以删除消息和队列

1. 找到“// ADD CODE TO DELETE MESSAGES AND THE QUEUE”注释，并在该注释后直接添加以下代码****。 请务必查看代码和注释。

    ```csharp
    // Delete messages from the queue with the DeleteMessagesAsync method.
    foreach (var message in (await queueClient.ReceiveMessagesAsync(maxMessages: 10)).Value)
    {
        // "Process" the message
        Console.WriteLine($"Deleting message: {message.MessageText}");
    
        // Let the service know we're finished with the message and it can be safely deleted.
        await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
    }
    Console.WriteLine("Messages deleted from the queue.");
    Console.WriteLine("\nPress Enter key to delete the queue...");
    Console.ReadLine();
    
    // Delete the queue with the DeleteAsync method.
    Console.WriteLine($"Deleting queue: {queueClient.Name}");
    await queueClient.DeleteAsync();
    
    Console.WriteLine("Done");
    ```

1. 按 Ctrl+S 保存文件，然后按 Ctrl+Q 退出编辑器********。

## 登录到 Azure 并运行应用

1. 在 Cloud Shell 命令行窗格中，输入以下命令以登录到 Azure。

    ```
    az login
    ```

    **<font color="red">必须登录到 Azure - 即使 Cloud Shell 会话已经过身份验证。</font>**

    > **备注**：在大多数情况下，仅使用 *az login* 就足够了。 但是，如果在多个租户中有订阅，则可能需要使用 *--tenant* 参数指定租户。 有关详细信息，请参阅[使用 Azure CLI 以交互方式登录到 Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)。

1. 运行以下命令启动控制台应用。 应用会在执行过程中会暂停多次，等待你按任意键继续。 这为你提供了一个在 Azure 门户中查看消息的机会。

    ```
    dotnet run
    ```

1. 在 Azure 门户中，导航到所创建的 Azure 存储帐户。 

1. 在左侧导航栏中展开“> 数据存储”，然后选择“队列”********。

1. 选择应用程序创建的队列，你可以查看已发送的消息，并监视应用程序正在执行的操作。

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。 

