---
lab:
  topic: Azure container services
  title: 使用 Azure 容器注册表任务生成并运行容器映像
  description: 了解如何使用 Azure CLI 命令通过Azure 容器注册表任务生成并运行容器映像。
---

# 使用 Azure 容器注册表任务生成并运行容器映像

在此练习中，将从应用程序代码生成容器映像，并使用 Azure CLI 将其推送到 Azure 容器注册表。 你将了解如何为容器化准备应用、创建 ACR 实例，以及将容器映像存储在 Azure 中。

在本练习中执行的任务：

* 创建 Azure 容器注册表资源
* 从 Dockerfile 生成和推送映像
* 验证结果
* 在 Azure 容器注册表中运行映像

此练习大约需要 20 分钟才能完成****。

## 创建 Azure 容器注册表资源

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***Bash*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。 如果系统提示你选择存储帐户来保存文件，请选择“不需要存储帐户”、你的订阅，然后选择“应用”********。

    > **备注**：如果以前创建了使用 *PowerShell* 环境的 Cloud Shell，请将其切换到 ***Bash***。

1. 为此练习所需的资源创建资源组。 将 myResourceGroup 替换为你希望在资源组中使用的名称****。 如果需要，可以将 useast 替换为附近的区域****。 如果已有要使用的资源组，请继续执行下一步。

    ```
    az group create --location eastus --name myResourceGroup
    ```

1. 运行以下命令来创建基本容器注册表。 注册表名称在 Azure 中必须唯一，并且包含 5-50 个字母数字字符。 将 myResourceGroup 替换为你之前使用过的名称，并将 myContainerRegistry 替换为唯一值********。

    ```bash
    az acr create --resource-group myResourceGroup \
        --name myContainerRegistry --sku Basic
    ```

    > **注意：** 命令会创建一个基本注册表，这是一个成本优化选项，供开发人员了解 Azure 容器注册表**。

## 从 Dockerfile 生成和推送映像

接下来，基于 Dockerfile 生成并推送映像。

1. 运行以下命令以创建 Dockerfile。 Dockerfile 包含单独的一行代码，用于引用在 Microsoft 容器注册表中托管的 hello-world 映像**。

    ```bash
    echo FROM mcr.microsoft.com/hello-world > Dockerfile
    ```

1. 运行以下 az acr build 命令，该命令将生成映像，并将在成功生成映像后将其推送到注册表****。 将 myContainerRegistry 替换为你之前创建的名称****。

    ```bash
    az acr build --image sample/hello-world:v1  \
        --registry myContainerRegistry \
        --file Dockerfile .
    ```

    下面是上一个命令输出的缩短示例，其中显示了最后几行和最终结果。 可以看到 repository 字段中列出了 sample/hello-word 映像****。

    ```
    - image:
        registry: myContainerRegistry.azurecr.io
        repository: sample/hello-world
        tag: v1
        digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
      runtime-dependency:
        registry: mcr.microsoft.com
        repository: hello-world
        tag: latest
        digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
      git: {}
    
    
    Run ID: cf1 was successful after 11s
    ```

## 验证结果

1. 运行以下命令，列出注册表中的存储库。 将 myContainerRegistry 替换为你之前创建的名称****。

    ```bash
    az acr repository list --name myContainerRegistry --output table
    ```

    输出：

    ```
    Result
    ----------------
    sample/hello-world
    ```

1. 运行以下命令，列出 sample/hello-world 存储库中的标记****。 将 myContainerRegistry 替换为你之前使用过的名称****。

    ```bash
    az acr repository show-tags --name myContainerRegistry \
        --repository sample/hello-world --output table
    ```

    输出：

    ```
    Result
    --------
    v1
    ```

## 运行 ACR 中的映像

1. 使用 az acr run 命令从容器注册表运行 sample/hello-world:v1 容器映像******。 以下示例使用 $Registry 指定运行命令的注册表****。 将 myContainerRegistry 替换为你之前使用过的名称****。

    ```bash
    az acr run --registry myContainerRegistry \
        --cmd '$Registry/sample/hello-world:v1' /dev/null
    ```

    此示例中的 cmd 参数会以其默认配置运行容器，但 cmd 支持其他 docker run 参数，甚至其他 docker 命令****************。 

    以下示例输出已缩短：

    ```
    Packing source code into tar to upload...
    Uploading archived source code from '/tmp/run_archive_ebf74da7fcb04683867b129e2ccad5e1.tar.gz'...
    Sending context (1.855 KiB) to registry: mycontainerre...
    Queued a run with ID: cab
    Waiting for an agent...
    2019/03/19 19:01:53 Using acb_vol_60e9a538-b466-475f-9565-80c5b93eaa15 as the home volume
    2019/03/19 19:01:53 Creating Docker network: acb_default_network, driver: 'bridge'
    2019/03/19 19:01:53 Successfully set up Docker network: acb_default_network
    2019/03/19 19:01:53 Setting up Docker configuration...
    2019/03/19 19:01:54 Successfully set up Docker configuration
    2019/03/19 19:01:54 Logging in to registry: mycontainerregistry008.azurecr.io
    2019/03/19 19:01:55 Successfully logged into mycontainerregistry008.azurecr.io
    2019/03/19 19:01:55 Executing step ID: acb_step_0. Working directory: '', Network: 'acb_default_network'
    2019/03/19 19:01:55 Launching container with name: acb_step_0
    
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    
    2019/03/19 19:01:56 Successfully executed container: acb_step_0
    2019/03/19 19:01:56 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 0.843801)
    
    Run ID: cab was successful after 6s
    ```

## 清理资源

完成练习后，应删除已创建的云资源，以避免不必要的资源使用。

1. 在浏览器中，导航到 Azure 门户，网址为：[https://portal.azure.com](https://portal.azure.com)；如果出现提示，请使用 Azure 凭据登录。
1. 导航到创建的资源组，然后查看本练习中使用的资源内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

> 注意：**** 删除资源组会删除其中包含的所有资源。 如果你为本练习选择了一个现有资源组，则该资源组中超出本练习范围的任何现有资源也将被一并删除。
