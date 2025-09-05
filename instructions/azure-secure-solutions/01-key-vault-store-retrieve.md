---
lab:
  topic: Secure solutions in Azure
  title: 从 Azure Key Vault 创建和检索机密
  description: 了解如何使用 Azure CLI 以编程方式创建密钥保管库，以及如何创建和检索机密。
---

# 从 Azure Key Vault 创建和检索机密

在此练习中，将创建 Azure 密钥保管库、使用 Azure CLI 存储机密，以及构建能从密钥保管库创建和检索机密的 .NET 控制台应用程序。 你将了解如何配置身份验证、以编程方式管理机密，以及如何在完成时清理资源。  

在本练习中执行的任务：

* 创建 Azure 密钥保管库资源
* 使用 Azure CLI 将机密存储在密钥保管库中
* 创建 .NET 控制台应用以创建和检索机密
* 清理资源

此练习大约需要 30 分钟才能完成****。

## 创建 Azure 密钥保管库资源并添加机密

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
    keyVaultName=mykeyvaultname$RANDOM
    ```

1. 运行以下命令以获取密钥保管库的名称并记录下来。 之后在练习中会用到它。

    ```
    echo $keyVaultName
    ```

1. 运行以下命令来创建一个 Azure 密钥保管库资源。 此操作需要几分钟时间。

    ```
    az keyvault create --name $keyVaultName \
        --resource-group $resourceGroup --location $location
    ```

### 将角色分配给 Microsoft Entra 用户名

要创建和检索机密，请将Microsoft Entra 用户分配到 “密钥保管库机密管理员”角色****。 这会授予用户帐户设置、删除和列出机密的权限。 在典型场景中，你可能希望将创建/读取操作分开：将“密钥保管库机密管理员”角色分配给一个组，将“密钥保管库机密用户”（可获取和列出机密）角色分配给另一个组********。

1. 运行以下命令，从你的帐户中检索 userPrincipalName****。 这代表该角色将被分配给的对象。

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. 运行以下命令以检索密钥保管库的资源 ID。 资源 ID 会将角色分配的范围限定为特定的密钥保管库。

    ```
    resourceID=$(az keyvault show --resource-group $resourceGroup \
        --name $keyVaultName --query id --output tsv)
    ```

1. 运行以下命令来创建并分配“密钥保管库机密管理员”角色****。

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Key Vault Secrets Officer" \
        --scope $resourceID
    ```

接下来，将机密添加到创建的密钥保管库。

### 使用 Azure CLI 添加和检索机密

1. 运行以下命令创建机密。 

    ```
    az keyvault secret set --vault-name $keyVaultName \
        --name "MySecret" --value "My secret value"
    ```

1. 运行以下命令检索该机密，验证它是否已设置成功。

    ```
    az keyvault secret show --name "MySecret" --vault-name $keyVaultName
    ```

    此命令返回一些 JSON。 最后一行包含纯文本格式的密码。 

    ```json
    "value": "My secret value"
    ```

## 创建 .NET 控制台应用以存储和检索机密

在将所需的资源部署到 Azure 后，下一步是设置控制台应用程序。 以下步骤均在 Cloud Shell 中执行。

>**
          **提示：通过拖动上边框调整 Cloud Shell 的大小以显示更多信息和代码。 还可以使用最小化和最大化按钮在 Cloud Shell 和主门户界面之间切换。

1. 运行以下命令创建一个目录来存放项目，并切换到该项目目录中。

    ```
    mkdir keyvault
    cd keyvault
    ```

1. 创建 .NET 控制台应用程序。

    ```
    dotnet new console
    ```

1. 运行以下命令，将 Azure.Identity 和 Azure.Security.KeyVault.Secret 包添加到项目********。

    ```
    dotnet add package Azure.Identity
    dotnet add package Azure.Security.KeyVault.Secrets
    ```

### 添加项目的起始代码

1. 在 Cloud Shell 中运行以下命令，以开始编辑应用程序。

    ```
    code Program.cs
    ```

1. 将任何现有内容替换为以下代码。 请确保将 YOUR-KEYVAULT-NAME 替换为实际密钥保管库的名称****。

    ```csharp
    using Azure.Identity;
    using Azure.Security.KeyVault.Secrets;
    
    // Replace YOUR-KEYVAULT-NAME with your actual Key Vault name
    string KeyVaultUrl = "https://YOUR-KEYVAULT-NAME.vault.azure.net/";
    
    
    // ADD CODE TO CREATE A CLIENT
    
    
    
    // ADD CODE TO CREATE A MENU SYSTEM
    
    
    
    // ADD CODE TO CREATE A SECRET
    
    
    
    // ADD CODE TO LIST SECRETS
    
    
    ```

1. 按 Ctrl+S 保存所做的更改****。

### 添加代码以完成应用程序

现在是时候添加代码来完成这个应用程序了。

1. 找到“// ADD CODE TO CREATE A CLIENT”注释，并在该注释后直接添加以下代码****。 请务必查看代码和注释。

    ```csharp
    // Configure authentication options for connecting to Azure Key Vault
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create the Key Vault client using the URL and authentication credentials
    var client = new SecretClient(new Uri(KeyVaultUrl), new DefaultAzureCredential(options));
    ```

1. 找到“// ADD CODE TO CREATE A MENU SYSTEM”注释，并在该注释后直接添加以下代码****。 请务必查看代码和注释。

    ```csharp
    // Main application loop - continues until user types 'quit'
    while (true)
    {
        // Display menu options to the user
        Console.Clear();
        Console.WriteLine("\nPlease select an option:");
        Console.WriteLine("1. Create a new secret");
        Console.WriteLine("2. List all secrets");
        Console.WriteLine("Type 'quit' to exit");
        Console.Write("Enter your choice: ");
    
        // Read user input and convert to lowercase for easier comparison
        string? input = Console.ReadLine()?.Trim().ToLower();
        
        // Check if user wants to exit the application
        if (input == "quit")
        {
            Console.WriteLine("Goodbye!");
            break;
        }
    
        // Process the user's menu selection
        switch (input)
        {
            case "1":
                // Call the method to create a new secret
                await CreateSecretAsync(client);
                break;
            case "2":
                // Call the method to list all existing secrets
                await ListSecretsAsync(client);
                break;
            default:
                // Handle invalid input
                Console.WriteLine("Invalid option. Please enter 1, 2, or 'quit'.");
                break;
        }
    }
    ```

1. 找到“// ADD CODE TO CREATE A SECRET”注释，并在该注释后直接添加以下代码****。 请务必查看代码和注释。

    ```csharp
    async Task CreateSecretAsync(SecretClient client)
    {
        try
        {
            Console.Clear();
            Console.WriteLine("\nCreating a new secret...");
            
            // Get the secret name from user input
            Console.Write("Enter secret name: ");
            string? secretName = Console.ReadLine()?.Trim();
    
            // Validate that the secret name is not empty
            if (string.IsNullOrEmpty(secretName))
            {
                Console.WriteLine("Secret name cannot be empty.");
                return;
            }
            
            // Get the secret value from user input
            Console.Write("Enter secret value: ");
            string? secretValue = Console.ReadLine()?.Trim();
    
            // Validate that the secret value is not empty
            if (string.IsNullOrEmpty(secretValue))
            {
                Console.WriteLine("Secret value cannot be empty.");
                return;
            }
    
            // Create a new KeyVaultSecret object with the provided name and value
            var secret = new KeyVaultSecret(secretName, secretValue);
            
            // Store the secret in Azure Key Vault
            await client.SetSecretAsync(secret);
    
            Console.WriteLine($"Secret '{secretName}' created successfully!");
            Console.WriteLine("Press Enter to continue...");
            Console.ReadLine();
        }
        catch (Exception ex)
        {
            // Handle any errors that occur during secret creation
            Console.WriteLine($"Error creating secret: {ex.Message}");
        }
    }
    ```

1. 找到“// ADD CODE TO LIST SECRETS”注释，并在该注释后直接添加以下代码****。 请务必查看代码和注释。

    ```csharp
    async Task ListSecretsAsync(SecretClient client)
    {
        try
        {
            Console.Clear();
            Console.WriteLine("Listing all secrets in the Key Vault...");
            Console.WriteLine("----------------------------------------");
    
            // Get an async enumerable of all secret properties in the Key Vault
            var secretProperties = client.GetPropertiesOfSecretsAsync();
            bool hasSecrets = false;
    
            // Iterate through each secret property to retrieve full secret details
            await foreach (var secretProperty in secretProperties)
            {
                hasSecrets = true;
                try
                {
                    // Retrieve the actual secret value and metadata using the secret name
                    var secret = await client.GetSecretAsync(secretProperty.Name);
                    
                    // Display the secret information to the console
                    Console.WriteLine($"Name: {secret.Value.Name}");
                    Console.WriteLine($"Value: {secret.Value.Value}");
                    Console.WriteLine($"Created: {secret.Value.Properties.CreatedOn}");
                    Console.WriteLine("----------------------------------------");
                }
                catch (Exception ex)
                {
                    // Handle errors for individual secrets (e.g., access denied, secret not found)
                    Console.WriteLine($"Error retrieving secret '{secretProperty.Name}': {ex.Message}");
                    Console.WriteLine("----------------------------------------");
                }
            }
    
            // Inform user if no secrets were found in the Key Vault
            if (!hasSecrets)
            {
                Console.WriteLine("No secrets found in the Key Vault.");
            }
        }
        catch (Exception ex)
        {
            // Handle general errors that occur during the listing operation
            Console.WriteLine($"Error listing secrets: {ex.Message}");
        
        }
        Console.WriteLine("Press Enter to continue...");
        Console.ReadLine();
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

1. 运行以下命令启动控制台应用。 该应用将显示应用程序的菜单系统。 

    ```
    dotnet run
    ```

1. 在本练习开始时你已创建了一个机密，请输入 2 以检索并显示该机密****。

1. 输入 1 并输入机密名称和值，以创建新机密****。

1. 再次列出机密以查看新添加的内容。

完成应用程序后，输入 quit****。

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。 
