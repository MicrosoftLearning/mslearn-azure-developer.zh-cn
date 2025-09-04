---
lab:
  topic: Azure Storage
  title: 使用 .NET 客户端库创建 Blob 存储资源
  description: 了解如何使用 Azure 存储 .NET 客户端库创建容器、上传和列出 blob 以及删除容器。
---

# 使用 .NET 客户端库创建 Blob 存储资源

在此练习中，你将创建一个 Azure 存储帐户，并使用 Azure 存储 Blob 客户端库构建一个 .NET 控制台应用程序，用于创建容器、将文件上传到 Blob 存储、列出 Blob 以及下载文件。 了解如何使用 Azure 进行身份验证、以编程方式执行 Blob 存储操作，以及如何在 Azure 门户中验证结果。

在本练习中执行的任务：

* 准备 Azure 资源
* 创建控制台应用以创建和下载数据
* 运行应用并验证结果
* 清理资源

此练习大约需要 30 分钟才能完成****。

## 创建 Azure 存储帐户

在练习的此部分，你将使用 Azure CLI 在 Azure 中创建所需的资源。

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***Bash*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。 如果系统提示你选择存储帐户来保存文件，请选择“不需要存储帐户”、你的订阅，然后选择“应用”********。

    > **备注**：如果以前创建了使用 *PowerShell* 环境的 Cloud Shell，请将其切换到 ***Bash***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

1. 为此练习所需的资源创建资源组。 将 myResourceGroup 替换为你希望在资源组中使用的名称****。 如果需要，可以将 eastus2 替换为附近的区域****。 如果已有要使用的资源组，请继续执行下一步。

    ```
    az group create --location eastus2 --name myResourceGroup
    ```

1. 许多命令都需要唯一的名称并使用相同的参数。 创建一些变量可以减少对创建资源的命令所需做的修改。 运行以下命令以创建所需的变量。 将 myResourceGroup 替换为你在本次练习中使用的名称****。

    ```
    resourceGroup=myResourceGroup
    location=eastus
    accountName=storageacct$RANDOM
    ```

1. 运行以下命令，创建 Azure 存储帐户，每个帐户名称必须唯一。 第一条命令会创建一个变量，该变量包含存储帐户的唯一名称。 从 echo 命令的输出结果中，记录下你的帐户名称****。 

    ```
    az storage account create --name $accountName \
        --resource-group $resourceGroup \
        --location $location \
        --sku Standard_LRS 
    
    echo $accountName
    ```

### 将角色分配给 Microsoft Entra 用户名

为了让你的应用能够创建资源和项，请将你的 Microsoft Entra 用户分配到“存储 Blob 数据所有者”角色****。 在 Cloud Shell 中执行以下步骤。

>**
          **提示：通过拖动上边框调整 Cloud Shell 的大小以显示更多信息和代码。 还可以使用最小化和最大化按钮在 Cloud Shell 和主门户界面之间切换。

1. 运行以下命令，从你的帐户中检索 userPrincipalName****。 这代表该角色将被分配给的对象。

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. 运行以下命令以检索存储帐户的资源 ID。 该资源 ID 会将角色分配的范围限定为特定的命名空间。

    ```
    resourceID=$(az storage account show --name $accountName \
        --resource-group $resourceGroup \
        --query id --output tsv)
    ```
1. 运行以下命令以创建和分配“存储 Blob 数据所有者”角色****。 此角色提供管理容器和项的权限。

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Storage Blob Data Owner" \
        --scope $resourceID
    ```

## 创建 .NET 控制台应用以创建容器和项

在将所需的资源部署到 Azure 后，下一步是设置控制台应用程序。 以下步骤均在 Cloud Shell 中执行。

1. 运行以下命令创建一个目录来存放项目，并切换到该项目目录中。

    ```
    mkdir azstor
    cd azstor
    ```

1. 创建 .NET 控制台应用程序。

    ```
    dotnet new console
    ```

1. 运行以下命令，在应用程序中添加所需的包。

    ```
    dotnet add package Azure.Storage.Blobs
    dotnet add package Azure.Identity
    ```

1. 运行以下命令，在项目中创建 data 文件夹****。 

    ```
    mkdir data
    ```

现在是时候为项目添加所需的代码了。

### 添加项目的起始代码

1. 在 Cloud Shell 中运行以下命令，以开始编辑应用程序。

    ```
    code Program.cs
    ```

1. 将任何现有内容替换为以下代码。 请务必查看代码中的注释。

    ```csharp
    using Azure.Storage.Blobs;
    using Azure.Storage.Blobs.Models;
    using Azure.Identity;
    
    Console.WriteLine("Azure Blob Storage exercise\n");
    
    // Create a DefaultAzureCredentialOptions object to configure the DefaultAzureCredential
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Run the examples asynchronously, wait for the results before proceeding
    await ProcessAsync();
    
    Console.WriteLine("\nPress enter to exit the sample application.");
    Console.ReadLine();
    
    async Task ProcessAsync()
    {
        // CREATE A BLOB STORAGE CLIENT
        
    
    
        // CREATE A CONTAINER
        
    
    
        // CREATE A LOCAL FILE FOR UPLOAD TO BLOB STORAGE
        
    
    
        // UPLOAD THE FILE TO BLOB STORAGE
        
    
    
        // LIST BLOBS IN THE CONTAINER
        
    
    
        // DOWNLOAD THE BLOB TO A LOCAL FILE
        
    
    }
    ```

1. 按 Ctrl+S 保存更改，并继续执行下一步操作****。


## 添加代码以完成项目

在本练习剩余的所有步骤中，请在指定区域添加代码，以创建完整的应用程序。 

1. 找到“// CREATE A BLOB STORAGE CLIENT”注释，然后直接在注释下方添加以下代码****。 BlobServiceClient 是管理存储帐户中容器和 Blob 的主要入口点****。 该客户端使用 DefaultAzureCredential 进行身份验证**。 请务必将 YOUR_ACCOUNT_NAME 替换为你之前记录的名称****。

    ```csharp
    // Create a credential using DefaultAzureCredential with configured options
    string accountName = "YOUR_ACCOUNT_NAME"; // Replace with your storage account name
    
    // Use the DefaultAzureCredential with the options configured at the top of the program
    DefaultAzureCredential credential = new DefaultAzureCredential(options);
    
    // Create the BlobServiceClient using the endpoint and DefaultAzureCredential
    string blobServiceEndpoint = $"https://{accountName}.blob.core.windows.net";
    BlobServiceClient blobServiceClient = new BlobServiceClient(new Uri(blobServiceEndpoint), credential);
    ```

1. 按 Ctrl+S 保存更改，并继续执行下一步操作****。

1. 找到“// CREATE A CONTAINER”注释，然后直接在注释下方添加以下代码****。 创建容器的步骤包括：创建 BlobServiceClient 类的实例，然后调用 CreateBlobContainerAsync 方法在存储帐户中创建容器********。 将 GUID 值追加到容器名称中，以保证其唯一性。 如果该容器已存在，则 CreateBlobContainerAsync 方法将失败****。

    ```csharp
    // Create a unique name for the container
    string containerName = "wtblob" + Guid.NewGuid().ToString();
    
    // Create the container and return a container client object
    Console.WriteLine("Creating container: " + containerName);
    BlobContainerClient containerClient = 
        await blobServiceClient.CreateBlobContainerAsync(containerName);
    
    // Check if the container was created successfully
    if (containerClient != null)
    {
        Console.WriteLine("Container created successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("Failed to create the container, exiting program.");
        return;
    }
    ```

1. 按 Ctrl+S 保存更改，并继续执行下一步操作****。

1. 找到“// CREATE A LOCAL FILE FOR UPLOAD TO BLOB STORAGE”注释，然后直接在该注释下添加以下代码****。 这会在数据目录中创建一个文件，该文件随后会被上传到容器中。

    ```csharp
    // Create a local file in the ./data/ directory for uploading and downloading
    Console.WriteLine("Creating a local file for upload to Blob storage...");
    string localPath = "./data/";
    string fileName = "wtfile" + Guid.NewGuid().ToString() + ".txt";
    string localFilePath = Path.Combine(localPath, fileName);
    
    // Write text to the file
    await File.WriteAllTextAsync(localFilePath, "Hello, World!");
    Console.WriteLine("Local file created, press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. 按 Ctrl+S 保存更改，并继续执行下一步操作****。

1. 找到“// UPLOAD THE FILE TO BLOB STORAGE”注释，然后直接在注释下方添加以下代码****。 该代码将通过对上一节中创建的容器调用 GetBlobClient 方法，获取对 BlobClient 对象的引用********。 然后，它会使用 UploadAsync 方法上传一个生成的本地文件****。 此方法将创建 Blob（如果该 Blob 尚不存在），或者覆盖 Blob（如果该 Blob 已存在）。

    ```csharp
    // Get a reference to the blob and upload the file
    BlobClient blobClient = containerClient.GetBlobClient(fileName);
    
    Console.WriteLine("Uploading to Blob storage as blob:\n\t {0}", blobClient.Uri);
    
    // Open the file and upload its data
    using (FileStream uploadFileStream = File.OpenRead(localFilePath))
    {
        await blobClient.UploadAsync(uploadFileStream);
        uploadFileStream.Close();
    }
    
    // Verify if the file was uploaded successfully
    bool blobExists = await blobClient.ExistsAsync();
    if (blobExists)
    {
        Console.WriteLine("File uploaded successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("File upload failed, exiting program..");
        return;
    }
    ```

1. 按 Ctrl+S 保存更改，并继续执行下一步操作****。

1. 找到“// LIST BLOBS IN THE CONTAINER”注释，然后直接在注释下方添加以下代码****。 通过调用 GetBlobsAsync 方法，列出容器中的 blob****。 在本示例中，仅将一个 Blob 添加到了容器中，因此列表操作仅返回该 Blob。 

    ```csharp
    Console.WriteLine("Listing blobs in container...");
    await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
    {
        Console.WriteLine("\t" + blobItem.Name);
    }
    
    Console.WriteLine("Press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. 按 Ctrl+S 保存更改，并继续执行下一步操作****。

1. 找到“// DOWNLOAD THE BLOB TO A LOCAL FILE”注释，然后直接在注释下方添加以下代码****。 该代码将通过使用 DownloadAsync 方法，将以前创建的 blob 下载到本地文件系统****。 示例代码向 Blob 名称添加了“DOWNLOADED”后缀，这样就可以在本地文件系统中看到这两个文件。 

    ```csharp
    // Adds the string "DOWNLOADED" before the .txt extension so it doesn't 
    // overwrite the original file
    
    string downloadFilePath = localFilePath.Replace(".txt", "DOWNLOADED.txt");
    
    Console.WriteLine("Downloading blob to: {0}", downloadFilePath);
    
    // Download the blob's contents and save it to a file
    BlobDownloadInfo download = await blobClient.DownloadAsync();
    
    using (FileStream downloadFileStream = File.OpenWrite(downloadFilePath))
    {
        await download.Content.CopyToAsync(downloadFileStream);
    }
    
    Console.WriteLine("Blob downloaded successfully to: {0}", downloadFilePath);
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

1. 在左侧导航栏中展开“> 数据存储”，然后选择“容器”********。

1. 选择应用程序创建的容器，可以查看已上传的 Blob。

1. 运行以下两条命令，切换到 data 目录并列出已上传和下载的文件****。

    ```
    cd data
    ls
    ```

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。 

