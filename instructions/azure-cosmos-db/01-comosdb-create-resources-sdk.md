---
lab:
  topic: Azure Cosmos DB
  title: 使用 .NET 在 Azure Cosmos DB for NoSQL 中创建资源
  description: 了解如何使用 Microsoft .NET SDK v3 在 Azure Cosmos DB 中创建数据库和容器资源。
---

# 使用 .NET 在 Azure Cosmos DB for NoSQL 中创建资源

在此练习中，你将创建 Azure Cosmos DB 帐户并生成 .NET 控制台应用程序，以使用 Microsoft Azure Cosmos DB SDK 创建数据库、容器和示例项。 你将了解如何配置身份验证、以编程方式执行数据库操作，以及如何在 Azure 门户中验证结果。

在本练习中执行的任务：

* 创建 Azure Cosmos DB 帐户
* 创建用于创建数据库、容器和项的控制台应用
* 运行控制台应用并验证结果

此练习大约需要 30 分钟才能完成****。

## 创建 Azure Cosmos DB 帐户

在练习的此部分，你将创建资源组和 Azure Cosmos DB 帐户。 还会记录帐户的终结点和访问密钥。

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***Bash*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。 如果系统提示你选择存储帐户来保存文件，请选择“不需要存储帐户”、你的订阅，然后选择“应用”********。

    > **备注**：如果以前创建了使用 *PowerShell* 环境的 Cloud Shell，请将其切换到 ***Bash***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

1. 为此练习所需的资源创建资源组。 如果已有要使用的资源组，请继续执行下一步。 将 myResourceGroup 替换为你希望在资源组中使用的名称****。 如果需要，可以将 useast 替换为附近的区域****。

    ```
    az group create --location eastus --name myResourceGroup
    ```

1. 许多命令都需要唯一的名称并使用相同的参数。 创建一些变量可以减少对创建资源的命令所需做的修改。 运行以下命令以创建所需的变量。 将 myResourceGroup 替换为你在本次练习中使用的名称****。

    ```
    resourceGroup=myResourceGroup
    accountName=cosmosexercise$RANDOM
    ```

1. 运行以下命令，创建 Azure Cosmos DB 帐户，每个帐户名称必须唯一。 

    ```
    az cosmosdb create --name $accountName \
        --resource-group $resourceGroup
    ```

1.  运行以下命令，检索 Azure Cosmos DB 帐户的 **documentEndpoint**。 记录命令结果中的终结点，稍后的练习中会用到它。

    ```
    az cosmosdb show --name $accountName \
        --resource-group $resourceGroup \
        --query "documentEndpoint" --output tsv
    ```

1. 使用下面的命令检索帐户的主键。 记录命令结果中的主键，稍后的练习中会用到它。

    ```
    az cosmosdb keys list --name $accountName \
        --resource-group $resourceGroup \
        --query "primaryMasterKey" --output tsv
    ```

## 使用 .NET 控制台应用程序创建数据资源和项

在将所需的资源部署到 Azure 后，下一步是设置控制台应用程序。 以下步骤均在 Cloud Shell 中执行。

>**
          **提示：通过拖动上边框调整 Cloud Shell 的大小以显示更多信息和代码。 还可以使用最小化和最大化按钮在 Cloud Shell 和主门户界面之间切换。

1. 为该项目创建一个文件夹，并更改到该文件夹。

    ```bash
    mkdir cosmosdb
    cd cosmosdb
    ```

1. 创建 .NET 控制台应用。

    ```bash
    dotnet new console
    ```

### 配置控制台应用程序

1. 运行以下命令，将 Microsoft.Azure.Cosmos、Newtonsoft.Json 和 dotenv.net 包添加到项目中************。

    ```bash
    dotnet add package Microsoft.Azure.Cosmos --version 3.*
    dotnet add package Newtonsoft.Json --version 13.*
    dotnet add package dotenv.net
    ```

1. 运行以下命令以创建 .env 文件来保存机密，然后在代码编辑器中打开该文件****。

    ```bash
    touch .env
    code .env
    ```

1. 将以下代码添加到 .env 文件****。 将 YOUR_DOCUMENT_ENDPOINT 和 YOUR_ACCOUNT_KEY 替换为你之前记录的值********。

    ```
    DOCUMENT_ENDPOINT="YOUR_DOCUMENT_ENDPOINT"
    ACCOUNT_KEY="YOUR_ACCOUNT_KEY"
    ```

1. 按 Ctrl+S 保存文件，然后按 Ctrl+Q 退出编辑器********。

现在，可以使用 Cloud Shell 中的编辑器替换 **Program.cs** 文件中的模板代码。

### 添加项目的起始代码

1. 在 Cloud Shell 中运行以下命令，以开始编辑应用程序。

    ```bash
    code Program.cs
    ```

1. 将任何现有代码替换为以下代码片段。 

    该代码提供了应用的整体结构。 查看代码中的注释，以了解其工作原理。 要完成应用程序，请在练习的后面部分添加指定区域中的代码。 

    ```csharp
    using Microsoft.Azure.Cosmos;
    using dotenv.net;
    
    string databaseName = "myDatabase"; // Name of the database to create or use
    string containerName = "myContainer"; // Name of the container to create or use
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    string cosmosDbAccountUrl = envVars["DOCUMENT_ENDPOINT"];
    string accountKey = envVars["ACCOUNT_KEY"];
    
    if (string.IsNullOrEmpty(cosmosDbAccountUrl) || string.IsNullOrEmpty(accountKey))
    {
        Console.WriteLine("Please set the DOCUMENT_ENDPOINT and ACCOUNT_KEY environment variables.");
        return;
    }
    
    // CREATE THE COSMOS DB CLIENT USING THE ACCOUNT URL AND KEY
    
    
    try
    {
        // CREATE A DATABASE IF IT DOESN'T ALREADY EXIST
    
    
        // CREATE A CONTAINER WITH A SPECIFIED PARTITION KEY
    
    
        // DEFINE A TYPED ITEM (PRODUCT) TO ADD TO THE CONTAINER
    
    
        // ADD THE ITEM TO THE CONTAINER
    
    
    }
    catch (CosmosException ex)
    {
        // Handle Cosmos DB-specific exceptions
        // Log the status code and error message for debugging
        Console.WriteLine($"Cosmos DB Error: {ex.StatusCode} - {ex.Message}");
    }
    catch (Exception ex)
    {
        // Handle general exceptions
        // Log the error message for debugging
        Console.WriteLine($"Error: {ex.Message}");
    }
    
    // This class represents a product in the Cosmos DB container
    public class Product
    {
        public string? id { get; set; }
        public string? name { get; set; }
        public string? description { get; set; }
    }
    ```

接下来，你将在项目的指定区域中添加代码，以创建客户端、数据库、容器，并向容器添加示例项。

### 添加代码，以创建客户端并执行操作 

1. 在“// CREATE THE COSMOS DB CLIENT USING THE ACCOUNT URL AND KEY”注释后的空白处，添加以下代码****。 此代码定义用于连接到 Azure Cosmos DB 帐户的客户端。

    ```csharp
    CosmosClient client = new(
        accountEndpoint: cosmosDbAccountUrl,
        authKeyOrResourceToken: accountKey
    );
    ```

    >备注：最佳做法是使用 *Azure 标识*库中的 **DefaultAzureCredential**。 这可能需要在 Azure 中进行一些额外的配置要求，具体取决于订阅的设置方式。 

1. 在“// CREATE A DATABASE IF IT DOESN'T ALREADY EXIST”注释后的空白处，添加以下代码****。 

    ```csharp
    Database database = await client.CreateDatabaseIfNotExistsAsync(databaseName);
    Console.WriteLine($"Created or retrieved database: {database.Id}");
    ```

1. 在“// CREATE A CONTAINER WITH A SPECIFIED PARTITION KEY”注释后的空白处，添加以下代码****。 

    ```csharp
    Container container = await database.CreateContainerIfNotExistsAsync(
        id: containerName,
        partitionKeyPath: "/id"
    );
    Console.WriteLine($"Created or retrieved container: {container.Id}");
    ```

1. 在“// DEFINE A TYPED ITEM (PRODUCT) TO ADD TO THE CONTAINER”注释后的空白处，添加以下代码****。 这将定义添加到容器中的项。

    ```csharp
    Product newItem = new Product
    {
        id = Guid.NewGuid().ToString(), // Generate a unique ID for the product
        name = "Sample Item",
        description = "This is a sample item in my Azure Cosmos DB exercise."
    };
    ```

1. 在“// ADD THE ITEM TO THE CONTAINER”注释后的空白处，添加以下代码****。 

    ```csharp
    ItemResponse<Product> createResponse = await container.CreateItemAsync(
        item: newItem,
        partitionKey: new PartitionKey(newItem.id)
    );

    Console.WriteLine($"Created item with ID: {createResponse.Resource.id}");
    Console.WriteLine($"Request charge: {createResponse.RequestCharge} RUs");
    ```

1. 请注意，代码完成后，保存进度时请使用 Ctrl + S 保存文件，然后使用 Ctrl + Q 退出编辑器********。

1. 在 Cloud Shell 中运行以下命令，测试项目中是否存在任何错误。 如果看到错误，请在编辑器中打开 *Program.cs* 文件，并检查是否有缺失代码或粘贴错误。

    ```
    dotnet build
    ```

现在项目已完成，可以开始运行应用程序并在 Azure 门户中验证结果。

## 运行应用程序并验证结果。

1. 在 Cloud Shell 中，运行 `dotnet run` 命令。 输出内容应类似于以下示例。

    ```
    Created or retrieved database: myDatabase
    Created or retrieved container: myContainer
    Created item: c549c3fa-054d-40db-a42b-c05deabbc4a6
    Request charge: 6.29 RUs
    ```

1. 在 Azure 门户中，导航到之前创建的 Azure Cosmos DB 资源。 在左侧导航栏中，选择“**数据资源管理器**”。 在“**数据资源管理器**”中，选择 **myDatabase**，然后展开 **myContainer**。 可以通过选择“**项**”来查看创建的项。

    ![显示“数据资源管理器”中“项”的位置的屏幕截图。](./media/01/cosmos-data-explorer.png)

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。
1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。
