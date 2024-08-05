---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/openai.ipynb
sidebar_label: OpenAI
---

# ChatOpenAI

本笔记本提供了有关如何开始使用 OpenAI [聊天模型](/docs/concepts/#chat-models) 的快速概述。有关所有 ChatOpenAI 特性和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.base.ChatOpenAI.html)。

OpenAI 有几个聊天模型。您可以在 [OpenAI 文档](https://platform.openai.com/docs/models) 中找到有关其最新模型及其成本、上下文窗口和支持的输入类型的信息。

:::info Azure OpenAI

请注意，某些 OpenAI 模型也可以通过 [Microsoft Azure 平台](https://azure.microsoft.com/en-us/products/ai-services/openai-service) 访问。要使用 Azure OpenAI 服务，请使用 [AzureChatOpenAI 集成](/docs/integrations/chat/azure_chat_openai/)。

:::

## 概述

### 集成详情
| 类 | 包 | 本地 | 可序列化 | [JS 支持](https://js.langchain.com/v0.2/docs/integrations/chat/openai) | 包下载 | 包最新 |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [ChatOpenAI](https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.base.ChatOpenAI.html) | [langchain-openai](https://api.python.langchain.com/en/latest/openai_api_reference.html) | ❌ | beta | ✅ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-openai?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain-openai?style=flat-square&label=%20) |

### 模型特性
| [工具调用](/docs/how_to/tool_calling) | [结构化输出](/docs/how_to/structured_output/) | JSON模式 | 图像输入 | 音频输入 | 视频输入 | [令牌级流式传输](/docs/how_to/chat_streaming/) | 原生异步 | [令牌使用](/docs/how_to/chat_token_usage_tracking/) | [Logprobs](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |

## 设置

要访问 OpenAI 模型，您需要创建一个 OpenAI 帐户，获取 API 密钥，并安装 `langchain-openai` 集成包。

### 凭证

前往 https://platform.openai.com 注册 OpenAI 并生成 API 密钥。完成后设置 OPENAI_API_KEY 环境变量：


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("Enter your OpenAI API key: ")
```
```output
Enter your OpenAI API key:  ········
```
如果您希望自动追踪模型调用，您还可以通过取消注释以下内容来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：


```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### 安装

LangChain OpenAI 集成位于 `langchain-openai` 包中：

```python
%pip install -qU langchain-openai
```

## 实例化

现在我们可以实例化我们的模型对象并生成聊天完成内容：

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4o",
    temperature=0,
    max_tokens=None,
    timeout=None,
    max_retries=2,
    # api_key="...",  # 如果您更喜欢直接传递 API 密钥而不是使用环境变量
    # base_url="...",
    # organization="...",
    # 其他参数...
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
AIMessage(content="J'adore la programmation.", response_metadata={'token_usage': {'completion_tokens': 5, 'prompt_tokens': 31, 'total_tokens': 36}, 'model_name': 'gpt-4o', 'system_fingerprint': 'fp_43dfabdef1', 'finish_reason': 'stop', 'logprobs': None}, id='run-012cffe2-5d3d-424d-83b5-51c6d4a593d1-0', usage_metadata={'input_tokens': 31, 'output_tokens': 5, 'total_tokens': 36})
```



```python
print(ai_msg.content)
```
```output
J'adore la programmation.
```

## 链接

我们可以使用提示模板来[链接](/docs/how_to/sequence/)我们的模型，如下所示：

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
AIMessage(content='Ich liebe Programmieren.', response_metadata={'token_usage': {'completion_tokens': 5, 'prompt_tokens': 26, 'total_tokens': 31}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': 'fp_b28b39ffa8', 'finish_reason': 'stop', 'logprobs': None}, id='run-94fa6741-c99b-4513-afce-c3f562631c79-0')
```

## 工具调用

OpenAI 有一个 [工具调用](https://platform.openai.com/docs/guides/function-calling)（我们在这里将“工具调用”和“函数调用”视为同义词）API，允许您描述工具及其参数，并让模型返回一个包含要调用的工具和该工具输入的 JSON 对象。工具调用对于构建工具使用链和代理非常有用，并且更一般地用于从模型获取结构化输出。

### ChatOpenAI.bind_tools()

通过 `ChatOpenAI.bind_tools`，我们可以轻松地将 Pydantic 类、字典模式、LangChain 工具，甚至函数作为工具传递给模型。在后台，这些会被转换为 OpenAI 工具模式，格式如下：
```
{
    "name": "...",
    "description": "...",
    "parameters": {...}  # JSONSchema
}
```
并在每次模型调用时传入。


```python
from langchain_core.pydantic_v1 import BaseModel, Field


class GetWeather(BaseModel):
    """获取给定地点的当前天气"""

    location: str = Field(..., description="城市和州，例如：旧金山，加州")


llm_with_tools = llm.bind_tools([GetWeather])
```


```python
ai_msg = llm_with_tools.invoke(
    "旧金山的天气如何",
)
ai_msg
```



```output
AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_H7fABDuzEau48T10Qn0Lsh0D', 'function': {'arguments': '{"location":"San Francisco"}', 'name': 'GetWeather'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 15, 'prompt_tokens': 70, 'total_tokens': 85}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': 'fp_b28b39ffa8', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-b469135e-2718-446a-8164-eef37e672ba2-0', tool_calls=[{'name': 'GetWeather', 'args': {'location': 'San Francisco'}, 'id': 'call_H7fABDuzEau48T10Qn0Lsh0D'}])
```

### AIMessage.tool_calls
请注意，AIMessage 有一个 `tool_calls` 属性。它包含以标准化的 ToolCall 格式表示的信息，且与模型提供者无关。

```python
ai_msg.tool_calls
```

```output
[{'name': 'GetWeather',
  'args': {'location': 'San Francisco'},
  'id': 'call_H7fABDuzEau48T10Qn0Lsh0D'}]
```

有关绑定工具和工具调用输出的更多信息，请访问 [tool calling](/docs/how_to/function_calling) 文档。

## 微调

您可以通过传入相应的 `modelName` 参数来调用经过微调的 OpenAI 模型。

这通常采用 `ft:{OPENAI_MODEL_NAME}:{ORG_NAME}::{MODEL_ID}` 的形式。例如：


```python
fine_tuned_model = ChatOpenAI(
    temperature=0, model_name="ft:gpt-3.5-turbo-0613:langchain::7qTVM5AR"
)

fine_tuned_model(messages)
```



```output
AIMessage(content="J'adore la programmation.", additional_kwargs={}, example=False)
```

## API 参考

有关所有 ChatOpenAI 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.base.ChatOpenAI.html

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)