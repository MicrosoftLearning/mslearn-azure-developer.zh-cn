---
lab:
  topic: Secure solutions in Azure
  title: 从 Azure 应用程序配置检索配置设置
  description: 了解如何创建 Azure 应用程序配置资源，并使用 Azure CLI 设置配置信息。 然后，使用 ConfigurationBuilder 检索应用程序的设置****。
---

# 从 Azure 应用程序配置检索配置设置

在此练习中，你将创建 Azure 应用程序配置资源、使用 Azure CLI 存储配置设置，以及构建一个 .NET 控制台应用程序以使用 ConfigurationBuilder 检索配置值****。 你将了解如何使用分层密钥组织设置，并验证应用程序以访问基于云的配置数据。

在本练习中执行的任务：

* 创建 Azure 应用程序配置资源
* 存储连接字符串配置信息
* 创建 .NET 控制台应用以检索配置信息
* 清理资源

此练习大约需要 15 分钟才能完成****。

## 创建 Azure 应用程序配置资源并添加配置信息

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
    appConfigName=appconfigname$RANDOM
    ```

1. 运行以下命令以获取应用程序配置资源的名称。 记录该名称，稍后在练习中需要用到它。

    ```
    echo $appConfigName
    ```

1. 运行以下命令，确保为订阅注册 Microsoft.AppConfiguration 提供程序****。

    ```
    az provider register --namespace Microsoft.AppConfiguration
    ```

1. 注册可能需要几分钟的时间才能完成。 运行以下命令以检查注册状态。 当结果返回“已注册”时，继续执行下一步****。

    ```
    az provider show --namespace Microsoft.AppConfiguration --query "registrationState"
    ```

1. 运行以下命令以创建 Azure 应用程序配置资源。 此操作需要几分钟时间。

    ```
    az appconfig create --location $location \
        --name $appConfigName \
        --resource-group $resourceGroup
        --sku Free
    ```

    >**
          **提示：如果因配额限制导致使用免费 SKU 无法创建 AppConfig 资源，请改用开发人员 SKU********。
    

### 将角色分配给 Microsoft Entra 用户名

要检索配置信息，需要将Microsoft Entra 用户分配到“应用程序配置数据读取者”角色****。 

1. 运行以下命令，从你的帐户中检索 userPrincipalName****。 这代表该角色将被分配给的对象。

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. 运行以下命令以检索应用程序配置服务的资源 ID。 该资源 ID 设置角色分配的范围。

    ```
    resourceID=$(az appconfig show --resource-group $resourceGroup \
        --name $appConfigName --query id --output tsv)
    ```

1. 运行以下命令以创建和分配“应用程序配置数据读取者”角色****。

    ```
    az role assignment create --assignee $userPrincipal \
        --role "App Configuration Data Reader" \
        --scope $resourceID
    ```

接下来，将占位符连接字符串添加到应用程序配置。

### 使用 Azure CLI 添加配置信息

在 Azure 应用程序配置中，像 Dev:conStr 这样的键属于分层键，也称为命名空间键****。 冒号 (:) 在此处充当分隔符，用于构建逻辑层次结构，其中：

* “Dev”代表命名空间或环境前缀（表明此配置用于开发环境）****
* “conStr”代表配置名称****

通过此分层结构，可以按环境、功能或应用程序组件来组织配置设置，从而更轻松地管理和检索相关设置。

运行以下命令来存储占位符连接字符串。 

```
az appconfig kv set --name $appConfigName \
    --key Dev:conStr \
    --value connectionString \
    --yes
```

此命令返回一些 JSON。 最后一行包含纯文本格式的值。 

```json
"value": "connectionString"
```

## 创建 .NET 控制台应用以检索配置信息

在将所需的资源部署到 Azure 后，下一步是设置控制台应用程序。 以下步骤均在 Cloud Shell 中执行。

>**
          **提示：通过拖动上边框调整 Cloud Shell 的大小以显示更多信息和代码。 还可以使用最小化和最大化按钮在 Cloud Shell 和主门户界面之间切换。

1. 运行以下命令创建一个目录来存放项目，并切换到该项目目录中。

    ```
    mkdir appconfig
    cd appconfig
    ```

1. 创建 .NET 控制台应用程序。

    ```
    dotnet new console
    ```

1. 运行以下命令，将 Azure.Identity 和 Microsoft.Extensions.Configuration.AzureAppConfiguration 包添加到项目********。

    ```
    dotnet add package Azure.Identity
    dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
    ```

### 添加项目所需的代码

1. 在 Cloud Shell 中运行以下命令，以开始编辑应用程序。

    ```
    code Program.cs
    ```

1. 将任何现有内容替换为以下代码。 请务必将 YOUR_APP_CONFIGURATION_NAME 替换为前面记录的名称，并通读代码中的注释****。

    ```csharp
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    using Azure.Identity;
    
    // Set the Azure App Configuration endpoint, replace YOUR_APP_CONFIGURATION_NAME
    // with the name of your actual App Configuration service
    
    string endpoint = "https://YOUR_APP_CONFIGURATION_NAME.azconfig.io"; 
    
    // Configure which authentication methods to use
    // DefaultAzureCredential tries multiple auth methods automatically
    DefaultAzureCredentialOptions credentialOptions = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create a configuration builder to combine multiple config sources
    var builder = new ConfigurationBuilder();
    
    // Add Azure App Configuration as a source
    // This connects to Azure and loads configuration values
    builder.AddAzureAppConfiguration(options =>
    {
        
        options.Connect(new Uri(endpoint), new DefaultAzureCredential(credentialOptions));
    });
    
    // Build the final configuration object
    try
    {
        var config = builder.Build();
        
        // Retrieve a configuration value by key name
        Console.WriteLine(config["Dev:conStr"]);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error connecting to Azure App Configuration: {ex.Message}");
    }
    ```

1. 按 Ctrl+S 保存文件，然后按 Ctrl+Q 退出编辑器********。

## 登录到 Azure 并运行应用

1. 在 Cloud Shell 中，输入以下命令登录到 Azure。

    ```
    az login
    ```

    **<font color="red">必须登录到 Azure - 即使已对 Cloud Shell 会话进行身份验证。</font>**

    > **备注**：在大多数情况下，仅使用 *az login* 就足够了。 但是，如果在多个租户中有订阅，则可能需要使用 *--tenant* 参数指定租户。 有关详细信息，请参阅[使用 Azure CLI 以交互方式登录到 Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)。

1. 运行以下命令启动控制台应用。 该应用将显示你在练习前面分配给 Dev:conStr 设置的 connectionString 值********。

    ```
    dotnet run
    ```

    该应用将显示你在练习前面分配给 Dev:conStr 设置的 connectionString 值********。

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。
1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。
