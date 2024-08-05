---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/anthropic.ipynb
sidebar_label: Anthropic
---

# ChatAnthropic

本笔记本提供了有关如何开始使用 Anthropic [聊天模型](/docs/concepts/#chat-models) 的快速概述。有关所有 ChatAnthropic 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/chat_models/langchain_anthropic.chat_models.ChatAnthropic.html)。

Anthropic 有几个聊天模型。您可以在 [Anthropic 文档](https://docs.anthropic.com/en/docs/models-overview) 中找到有关其最新模型及其成本、上下文窗口和支持的输入类型的信息。

:::info AWS Bedrock 和 Google VertexAI

请注意，某些 Anthropic 模型也可以通过 AWS Bedrock 和 Google VertexAI 访问。请参阅 [ChatBedrock](/docs/integrations/chat/bedrock/) 和 [ChatVertexAI](/docs/integrations/chat/google_vertex_ai_palm/) 集成，以通过这些服务使用 Anthropic 模型。

:::

## 概述

### 集成细节

| 类 | 包 | 本地 | 可序列化 | [JS 支持](https://js.langchain.com/v0.2/docs/integrations/chat/anthropic) | 包下载量 | 包最新版本 |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [ChatAnthropic](https://api.python.langchain.com/en/latest/chat_models/langchain_anthropic.chat_models.ChatAnthropic.html) | [langchain-anthropic](https://api.python.langchain.com/en/latest/anthropic_api_reference.html) | ❌ | beta | ✅ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-anthropic?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain-anthropic?style=flat-square&label=%20) |

### 模型特性
| [工具调用](/docs/how_to/tool_calling) | [结构化输出](/docs/how_to/structured_output/) | JSON 模式 | [图像输入](/docs/how_to/multimodal_inputs/) | 音频输入 | 视频输入 | [令牌级流式传输](/docs/how_to/chat_streaming/) | 原生异步 | [令牌使用](/docs/how_to/chat_token_usage_tracking/) | [Logprobs](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ❌ |

## 设置

要访问 Anthropic 模型，您需要创建一个 Anthropic 账户，获取一个 API 密钥，并安装 `langchain-anthropic` 集成包。

### 凭证

前往 https://console.anthropic.com/ 注册 Anthropic 并生成 API 密钥。完成后设置 ANTHROPIC_API_KEY 环境变量：


```python
import getpass
import os

os.environ["ANTHROPIC_API_KEY"] = getpass.getpass("Enter your Anthropic API key: ")
```

如果您想要自动跟踪模型调用，可以通过取消注释以下内容来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：


```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### 安装

LangChain Anthropic 集成位于 `langchain-anthropic` 包中：


```python
%pip install -qU langchain-anthropic
```

## 实例化

现在我们可以实例化我们的模型对象并生成聊天完成内容：


```python
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(
    model="claude-3-5-sonnet-20240620",
    temperature=0,
    max_tokens=1024,
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
AIMessage(content="J'adore la programmation.", response_metadata={'id': 'msg_018Nnu76krRPq8HvgKLW4F8T', 'model': 'claude-3-5-sonnet-20240620', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 29, 'output_tokens': 11}}, id='run-57e9295f-db8a-48dc-9619-babd2bedd891-0', usage_metadata={'input_tokens': 29, 'output_tokens': 11, 'total_tokens': 40})
```



```python
print(ai_msg.content)
```
```output
J'adore la programmation.
```

## 链接

我们可以通过一个提示模板来[链接](/docs/how_to/sequence/)我们的模型，如下所示：


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
AIMessage(content="Here's the German translation:\n\nIch liebe Programmieren.", response_metadata={'id': 'msg_01GhkRtQZUkA5Ge9hqmD8HGY', 'model': 'claude-3-5-sonnet-20240620', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 23, 'output_tokens': 18}}, id='run-da5906b4-b200-4e08-b81a-64d4453643b6-0', usage_metadata={'input_tokens': 23, 'output_tokens': 18, 'total_tokens': 41})
```

## 内容块

一个需要注意的关键区别是，Anthropic模型与大多数其他模型之间的不同在于，单个Anthropic AI消息的内容可以是一个字符串或**内容块列表**。例如，当Anthropic模型调用工具时，工具调用是消息内容的一部分（同时也在标准化的`AIMessage.tool_calls`中暴露）：

```python
from langchain_core.pydantic_v1 import BaseModel, Field


class GetWeather(BaseModel):
    """获取给定位置的当前天气"""

    location: str = Field(..., description="城市和州，例如：旧金山，加州")


llm_with_tools = llm.bind_tools([GetWeather])
ai_msg = llm_with_tools.invoke("今天哪个城市更热：洛杉矶还是纽约？")
ai_msg.content
```



```output
[{'text': "要回答这个问题，我们需要检查洛杉矶（LA）和纽约（NY）的当前天气。我将使用GetWeather函数来获取这两个城市的信息。",
  'type': 'text'},
 {'id': 'toolu_01Ddzj5PkuZkrjF4tafzu54A',
  'input': {'location': 'Los Angeles, CA'},
  'name': 'GetWeather',
  'type': 'tool_use'},
 {'id': 'toolu_012kz4qHZQqD4qg8sFPeKqpP',
  'input': {'location': 'New York, NY'},
  'name': 'GetWeather',
  'type': 'tool_use'}]
```



```python
ai_msg.tool_calls
```



```output
[{'name': 'GetWeather',
  'args': {'location': 'Los Angeles, CA'},
  'id': 'toolu_01Ddzj5PkuZkrjF4tafzu54A'},
 {'name': 'GetWeather',
  'args': {'location': 'New York, NY'},
  'id': 'toolu_012kz4qHZQqD4qg8sFPeKqpP'}]
```

## API 参考

有关所有 ChatAnthropic 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/chat_models/langchain_anthropic.chat_models.ChatAnthropic.html

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)