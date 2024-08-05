---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/azure_openai.ipynb
---

# Azure OpenAI

:::caution
您当前正在查看关于使用 Azure OpenAI [文本完成模型](/docs/concepts/#llms) 的页面。最新和最受欢迎的 Azure OpenAI 模型是 [聊天完成模型](/docs/concepts/#chat-models)。

除非您正在特别使用 `gpt-3.5-turbo-instruct`，否则您可能更想查看 [此页面](/docs/integrations/chat/azure_chat_openai/)。
:::

本页面介绍如何将 LangChain 与 [Azure OpenAI](https://aka.ms/azure-openai) 一起使用。

Azure OpenAI API 与 OpenAI 的 API 兼容。`openai` Python 包使得同时使用 OpenAI 和 Azure OpenAI 变得简单。您可以以与调用 OpenAI 相同的方式调用 Azure OpenAI，以下是注意事项。

## API 配置
您可以通过环境变量配置 `openai` 包以使用 Azure OpenAI。以下是针对 `bash` 的配置：

```bash
# 您想要使用的 API 版本：将其设置为 `2023-12-01-preview` 以使用发布版本。
export OPENAI_API_VERSION=2023-12-01-preview
# 您的 Azure OpenAI 资源的基础 URL。您可以在 Azure 门户的 Azure OpenAI 资源下找到此信息。
export AZURE_OPENAI_ENDPOINT=https://your-resource-name.openai.azure.com
# 您的 Azure OpenAI 资源的 API 密钥。您可以在 Azure 门户的 Azure OpenAI 资源下找到此信息。
export AZURE_OPENAI_API_KEY=<your Azure OpenAI API key>
```

或者，您可以直接在运行的 Python 环境中配置 API：

```python
import os
os.environ["OPENAI_API_VERSION"] = "2023-12-01-preview"
```

## Azure Active Directory 认证
您可以通过两种方式对 Azure OpenAI 进行认证：
- API 密钥
- Azure Active Directory (AAD)

使用 API 密钥是开始的最简单方法。您可以在 Azure 门户的 Azure OpenAI 资源下找到您的 API 密钥。

但是，如果您有复杂的安全需求，您可能希望使用 Azure Active Directory。您可以在 [这里](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/managed-identity) 找到有关如何将 AAD 与 Azure OpenAI 一起使用的更多信息。

如果您在本地开发，您需要安装 Azure CLI 并登录。您可以在 [这里](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) 安装 Azure CLI。然后，运行 `az login` 进行登录。

为 Azure OpenAI 资源添加一个角色的 Azure 角色分配 `Cognitive Services OpenAI User`。这将允许您从 AAD 获取一个令牌以用于 Azure OpenAI。您可以将此角色分配授予用户、组、服务主体或托管身份。有关 Azure OpenAI RBAC 角色的更多信息，请参见 [这里](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/role-based-access-control)。

要在 Python 中使用 AAD 和 LangChain，请安装 `azure-identity` 包。然后，将 `OPENAI_API_TYPE` 设置为 `azure_ad`。接下来，使用 `DefaultAzureCredential` 类通过调用 `get_token` 从 AAD 获取令牌，如下所示。最后，将 `OPENAI_API_KEY` 环境变量设置为令牌值。

```python
import os
from azure.identity import DefaultAzureCredential

# Get the Azure Credential
credential = DefaultAzureCredential()

# Set the API type to `azure_ad`
os.environ["OPENAI_API_TYPE"] = "azure_ad"
# Set the API_KEY to the token from the Azure credential
os.environ["OPENAI_API_KEY"] = credential.get_token("https://cognitiveservices.azure.com/.default").token
```

`DefaultAzureCredential` 类是开始使用 AAD 认证的简单方法。如果需要，您还可以自定义凭据链。在下面的示例中，我们首先尝试使用托管身份，然后回退到 Azure CLI。这在您在 Azure 中运行代码但希望在本地开发时非常有用。

```python
from azure.identity import ChainedTokenCredential, ManagedIdentityCredential, AzureCliCredential

credential = ChainedTokenCredential(
    ManagedIdentityCredential(),
    AzureCliCredential()
)
```

## 部署
使用 Azure OpenAI，您可以设置自己的常见 GPT-3 和 Codex 模型的部署。在调用 API 时，您需要指定要使用的部署。

_**注意**：这些文档适用于 Azure 文本补全模型。像 GPT-4 这样的模型是聊天模型。它们具有稍微不同的接口，可以通过 `AzureChatOpenAI` 类访问。有关 Azure 聊天的文档，请参见 [Azure Chat OpenAI documentation](/docs/integrations/chat/azure_chat_openai)。_

假设您的部署名称是 `gpt-35-turbo-instruct-prod`。在 `openai` Python API 中，您可以使用 `engine` 参数指定此部署。例如：

```python
import openai

client = AzureOpenAI(
    api_version="2023-12-01-preview",
)

response = client.completions.create(
    model="gpt-35-turbo-instruct-prod",
    prompt="Test prompt"
)
```



```python
%pip install --upgrade --quiet  langchain-openai
```


```python
import os

os.environ["OPENAI_API_VERSION"] = "2023-12-01-preview"
os.environ["AZURE_OPENAI_ENDPOINT"] = "..."
os.environ["AZURE_OPENAI_API_KEY"] = "..."
```


```python
# 导入 Azure OpenAI
from langchain_openai import AzureOpenAI
```


```python
# 创建 Azure OpenAI 的实例
# 将部署名称替换为您自己的
llm = AzureOpenAI(
    deployment_name="gpt-35-turbo-instruct-0914",
)
```


```python
# 运行 LLM
llm.invoke("Tell me a joke")
```



```output
" Why couldn't the bicycle stand up by itself?\n\nBecause it was two-tired!"
```


我们也可以打印 LLM 并查看其自定义打印内容。


```python
print(llm)
```
```output
[1mAzureOpenAI[0m
Params: {'deployment_name': 'gpt-35-turbo-instruct-0914', 'model_name': 'gpt-3.5-turbo-instruct', 'temperature': 0.7, 'top_p': 1, 'frequency_penalty': 0, 'presence_penalty': 0, 'n': 1, 'logit_bias': {}, 'max_tokens': 256}
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)