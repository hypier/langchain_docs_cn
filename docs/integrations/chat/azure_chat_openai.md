---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/azure_chat_openai.ipynb
sidebar_label: Azure OpenAI
---

# AzureChatOpenAI

本指南将帮助您开始使用 AzureOpenAI [聊天模型](/docs/concepts/#chat-models)。有关所有 AzureChatOpenAI 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.azure.AzureChatOpenAI.html)。

Azure OpenAI 有多个聊天模型。您可以在 [Azure 文档](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models) 中找到有关其最新模型、成本、上下文窗口和支持的输入类型的信息。

:::info Azure OpenAI 与 OpenAI

Azure OpenAI 是指托管在 [Microsoft Azure 平台](https://azure.microsoft.com/en-us/products/ai-services/openai-service) 上的 OpenAI 模型。OpenAI 还提供自己的模型 API。要直接访问 OpenAI 服务，请使用 [ChatOpenAI 集成](/docs/integrations/chat/openai/)。

:::

## 概述

### 集成细节

| 类别 | 包 | 本地 | 可序列化 | [JS 支持](https://js.langchain.com/v0.2/docs/integrations/chat/azure) | 包下载 | 包最新 |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [AzureChatOpenAI](https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.azure.AzureChatOpenAI.html) | [langchain-openai](https://api.python.langchain.com/en/latest/openai_api_reference.html) | ❌ | beta | ✅ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-openai?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain-openai?style=flat-square&label=%20) |

### 模型特性
| [工具调用](/docs/how_to/tool_calling) | [结构化输出](/docs/how_to/structured_output/) | JSON模式 | [图像输入](/docs/how_to/multimodal_inputs/) | 音频输入 | 视频输入 | [令牌级流式传输](/docs/how_to/chat_streaming/) | 原生异步 | [令牌使用](/docs/how_to/chat_token_usage_tracking/) | [Logprobs](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |

## 设置

要访问 AzureOpenAI 模型，您需要创建一个 Azure 账户，创建一个 Azure OpenAI 模型的部署，获取您的部署名称和终端，获取 Azure OpenAI API 密钥，并安装 `langchain-openai` 集成包。

### 凭证

前往 [Azure 文档](https://learn.microsoft.com/en-us/azure/ai-services/openai/chatgpt-quickstart?tabs=command-line%2Cpython-new&pivots=programming-language-python) 创建您的部署并生成 API 密钥。完成后，设置 AZURE_OPENAI_API_KEY 和 AZURE_OPENAI_ENDPOINT 环境变量：

```python
import getpass
import os

os.environ["AZURE_OPENAI_API_KEY"] = getpass.getpass("Enter your AzureOpenAI API key: ")
os.environ["AZURE_OPENAI_ENDPOINT"] = "https://YOUR-ENDPOINT.openai.azure.com/"
```

如果您想获取模型调用的自动追踪，可以通过取消注释以下内容来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：

```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### 安装

LangChain AzureOpenAI 集成位于 `langchain-openai` 包中：

```python
%pip install -qU langchain-openai
```

## 实例化

现在我们可以实例化我们的模型对象并生成聊天补全。
- 将 `azure_deployment` 替换为您的部署名称，
- 您可以在此处找到最新支持的 `api_version`: https://learn.microsoft.com/en-us/azure/ai-services/openai/reference.


```python
from langchain_openai import AzureChatOpenAI

llm = AzureChatOpenAI(
    azure_deployment="gpt-35-turbo",  # or your deployment
    api_version="2023-06-01-preview",  # or your api version
    temperature=0,
    max_tokens=None,
    timeout=None,
    max_retries=2,
    # other params...
)
```

## 调用


```python
messages = [
    (
        "system",
        "You are a helpful assistant that translates English to French. Translate the user sentence.",
    ),
    ("human", "I love programming."),
]
ai_msg = llm.invoke(messages)
ai_msg
```



```output
AIMessage(content="J'adore la programmation.", response_metadata={'token_usage': {'completion_tokens': 8, 'prompt_tokens': 31, 'total_tokens': 39}, 'model_name': 'gpt-35-turbo', 'system_fingerprint': None, 'prompt_filter_results': [{'prompt_index': 0, 'content_filter_results': {'hate': {'filtered': False, 'severity': 'safe'}, 'self_harm': {'filtered': False, 'severity': 'safe'}, 'sexual': {'filtered': False, 'severity': 'safe'}, 'violence': {'filtered': False, 'severity': 'safe'}}}], 'finish_reason': 'stop', 'logprobs': None, 'content_filter_results': {'hate': {'filtered': False, 'severity': 'safe'}, 'self_harm': {'filtered': False, 'severity': 'safe'}, 'sexual': {'filtered': False, 'severity': 'safe'}, 'violence': {'filtered': False, 'severity': 'safe'}}}, id='run-bea4b46c-e3e1-4495-9d3a-698370ad963d-0', usage_metadata={'input_tokens': 31, 'output_tokens': 8, 'total_tokens': 39})
```



```python
print(ai_msg.content)
```
```output
J'adore la programmation.
```

## 链接

我们可以用提示模板来[链接](/docs/how_to/sequence/)我们的模型，如下所示：

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant that translates {input_language} to {output_language}.",
        ),
        ("human", "{input}"),
    ]
)

chain = prompt | llm
chain.invoke(
    {
        "input_language": "English",
        "output_language": "German",
        "input": "I love programming.",
    }
)
```



```output
AIMessage(content='Ich liebe das Programmieren.', response_metadata={'token_usage': {'completion_tokens': 6, 'prompt_tokens': 26, 'total_tokens': 32}, 'model_name': 'gpt-35-turbo', 'system_fingerprint': None, 'prompt_filter_results': [{'prompt_index': 0, 'content_filter_results': {'hate': {'filtered': False, 'severity': 'safe'}, 'self_harm': {'filtered': False, 'severity': 'safe'}, 'sexual': {'filtered': False, 'severity': 'safe'}, 'violence': {'filtered': False, 'severity': 'safe'}}}], 'finish_reason': 'stop', 'logprobs': None, 'content_filter_results': {'hate': {'filtered': False, 'severity': 'safe'}, 'self_harm': {'filtered': False, 'severity': 'safe'}, 'sexual': {'filtered': False, 'severity': 'safe'}, 'violence': {'filtered': False, 'severity': 'safe'}}}, id='run-cbc44038-09d3-40d4-9da2-c5910ee636ca-0', usage_metadata={'input_tokens': 26, 'output_tokens': 6, 'total_tokens': 32})
```

## 指定模型版本

Azure OpenAI 的响应包含 `model_name` 响应元数据属性，它是用于生成响应的模型名称。然而，与原生 OpenAI 响应不同，它不包含在 Azure 部署中设置的模型的具体版本。例如，它无法区分 `gpt-35-turbo-0125` 和 `gpt-35-turbo-0301`。这使得很难知道生成响应所使用的模型版本，这可能导致例如使用 `OpenAICallbackHandler` 时的总成本计算错误。

为了解决这个问题，您可以将 `model_version` 参数传递给 `AzureChatOpenAI` 类，该参数将被添加到 llm 输出中的模型名称中。通过这种方式，您可以轻松区分不同版本的模型。

```python
%pip install -qU langchain-community
```

```python
from langchain_community.callbacks import get_openai_callback

with get_openai_callback() as cb:
    llm.invoke(messages)
    print(
        f"Total Cost (USD): ${format(cb.total_cost, '.6f')}"
    )  # 在未指定模型版本的情况下，使用每 1k 输入和输出令牌 0.002 美元的固定费率
```
```output
Total Cost (USD): $0.000063
```

```python
llm_0301 = AzureChatOpenAI(
    azure_deployment="gpt-35-turbo",  # 或您的部署
    api_version="2023-06-01-preview",  # 或您的 API 版本
    model_version="0301",
)
with get_openai_callback() as cb:
    llm_0301.invoke(messages)
    print(f"Total Cost (USD): ${format(cb.total_cost, '.6f')}")
```
```output
Total Cost (USD): $0.000074
```

## API 参考

有关所有 AzureChatOpenAI 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.azure.AzureChatOpenAI.html

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)