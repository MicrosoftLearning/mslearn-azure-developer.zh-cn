---
lab:
  topic: Azure authentication and authorization
  title: 使用 MSAL.NET 实现交互式身份验证
  description: 了解如何使用 MSAL.NET SDK 实现交互式身份验证并获取令牌。
---

# 使用 MSAL.NET 实现交互式身份验证

在此练习中，将在 Microsoft Entra ID 中注册一个应用程序，然后创建一个 .NET 控制台应用程序，以使用 MSAL.NET 进行交互式身份验证并获取 Microsoft Graph 的访问令牌。 你需要了解如何配置身份验证范围、处理用户同意，以及如何缓存令牌以供后续运行。 

在本练习中执行的任务：

* 将应用程序注册到 Microsoft 标识平台
* 创建 .NET 控制台应用，实现 PublicClientApplicationBuilder 类以配置身份验证****。
* 使用 user.read Microsoft Graph 权限以交互方式获取令牌****。

此练习大约需要 15 分钟才能完成****。

## 开始之前

要完成此练习，需要满足以下先决条件：

* Azure 订阅。 如果你还没有，可以[注册一个](https://azure.microsoft.com/)。

* 安装在某个[受支持的平台](https://code.visualstudio.com/docs/supporting/requirements#_platforms)上的 [Visual Studio Code](https://code.visualstudio.com/)。

* [.NET 8](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) 或更高版本。

* 适用于 Visual Studio Code 的 [C# 开发工具包](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit)。

## 注册新应用程序

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。

1. 在门户中，搜索并选择“应用注册”****。 

1. 选择“+ 新建注册”，显示“注册应用程序”页面时，输入应用程序的注册信息********：

    | 字段 | 值 |
    |--|--|
    | **Name** | 输入 `myMsalApplication`  |
    | **支持的帐户类型** | 选择“仅此组织目录中的帐户” |
    | **重定向 URI（可选）** | 选择“公共客户端/本机(移动和桌面)”并在右侧框中输入 `http://localhost`。**** |

1. 选择“注册”。 Microsoft Entra ID 将为你的应用分配一个唯一的应用程序（客户端）ID，然后转至应用程序的“概览”**** 页面。 

1. 在“概述”页的“基本信息”部分，记录“应用程序(客户端) ID”和“目录(租户) ID”****************。 这些信息是应用程序所必需的。

    ![显示要复制的字段位置的屏幕截图。](./media/01-app-directory-id-location.png)
 
## 创建 .NET 控制台应用以获取令牌

在将所需的资源部署到 Azure 后，下一步是设置控制台应用程序。 在本地环境中执行以下步骤。

1. 创建名为 authapp（或自定义名称）的文件夹以存放项目****。

1. 启动 Visual Studio Code 并选择“文件”>“打开文件夹...”，然后选择项目文件夹********。

1. 选择“查看”>“终端”以打开终端窗口****。

1. 在 VS Code 终端中运行以下命令以创建 .NET 控制台应用程序。

    ```
    dotnet new console
    ```

1. 运行以下命令，将 Microsoft.Identity.Client 和 dotenv.net 包添加到项目********。

    ```
    dotnet add package Microsoft.Identity.Client
    dotnet add package dotenv.net
    ```

### 配置控制台应用程序

在本部分，将创建并编辑一个 .env 文件，用于保存你之前记录的机密****。 

1. 选择“文件”>“新建文件...”，在项目文件夹中创建名为 .env 的文件******。

1. 打开 .env 文件，并添加以下代码****。 将 YOUR_CLIENT_ID 和 YOUR_TENANT_ID 替换为你之前记录的值********。

    ```
    CLIENT_ID="YOUR_CLIENT_ID"
    TENANT_ID="YOUR_TENANT_ID"
    ```

1. 按 Ctrl+S 保存所做的更改****。

### 添加项目的起始代码

1. 打开 Program.cs 文件，将任何现有内容替换为以下代码**。 请务必查看代码中的注释。

    ```csharp
    using Microsoft.Identity.Client;
    using dotenv.net;
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    
    // Retrieve Azure AD Application ID and tenant ID from environment variables
    string _clientId = envVars["CLIENT_ID"];
    string _tenantId = envVars["TENANT_ID"];
    
    // ADD CODE TO DEFINE SCOPES AND CREATE CLIENT 
    
    
    
    // ADD CODE TO ACQUIRE AN ACCESS TOKEN
    
    
    ```

1. 按 Ctrl+S 保存所做的更改****。

### 添加代码以完成应用程序

1. 找到“// ADD CODE TO DEFINE SCOPES AND CREATE CLIENT”注释，并在该注释后直接添加以下代码****。 请务必查看代码中的注释。

    ```csharp
    // Define the scopes required for authentication
    string[] _scopes = { "User.Read" };
    
    // Build the MSAL public client application with authority and redirect URI
    var app = PublicClientApplicationBuilder.Create(_clientId)
        .WithAuthority(AzureCloudInstance.AzurePublic, _tenantId)
        .WithDefaultRedirectUri()
        .Build();
    ```

1. 找到“// ADD CODE TO ACQUIRE AN ACCESS TOKEN”注释，并在该注释后直接添加以下代码****。 请务必查看代码中的注释。

    ```csharp
    // Attempt to acquire an access token silently or interactively
    AuthenticationResult result;
    try
    {
        // Try to acquire token silently from cache for the first available account
        var accounts = await app.GetAccountsAsync();
        result = await app.AcquireTokenSilent(_scopes, accounts.FirstOrDefault())
                    .ExecuteAsync();
    }
    catch (MsalUiRequiredException)
    {
        // If silent token acquisition fails, prompt the user interactively
        result = await app.AcquireTokenInteractive(_scopes)
                    .ExecuteAsync();
    }
    
    // Output the acquired access token to the console
    Console.WriteLine($"Access Token:\n{result.AccessToken}");
    ```

1. 按 Ctrl+S 保存文件，然后按 Ctrl+Q 退出编辑器********。

## 运行应用程序

完成该应用后，接下来便可运行该应用。 

1. 通过运行以下命令启动应用程序：

    ```
    dotnet run
    ```

1. 应用将打开默认浏览器，提示你选择要进行身份验证的帐户。 如果列出了多个帐户，请选择与应用中使用的租户关联的那个帐户。

1. 如果你是第一次向已注册的应用进行身份验证，你将收到一条“请求的权限”通知，要求你批准该应用对你进行登录、读取你的个人资料，以及维护对已授予访问权限的数据的访问权限****。 选择“接受”。

    ![显示“请求征得的许可”通知的屏幕截图](./media/01-granting-permission.png)

1. 你应该会在控制台中看到与以下示例类似的结果。

    ```
    Access Token:
    eyJ0eXAiOiJKV1QiLCJub25jZSI6IlZF.........
    ```

1. 再次启动应用程序，你会发现不再收到 “请求的权限”通知****。 之前授予的权限已缓存。

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。
1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。
