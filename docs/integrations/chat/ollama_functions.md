---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/ollama_functions.ipynb
sidebar_label: Ollama 函数
sidebar_class_name: hidden
---

# OllamaFunctions

:::warning

这是一个实验性的包装器，旨在为不原生支持工具调用的模型添加工具调用支持。现在的 [Ollama 集成](/docs/integrations/chat/ollama/) 已经支持工具调用，应该使用它。

:::
本笔记本展示了如何使用一个围绕 Ollama 的实验性包装器，使其具备 [工具调用能力](https://python.langchain.com/v0.2/docs/concepts/#functiontool-calling)。

请注意，更强大和更具能力的模型在处理复杂模式和/或多个函数时表现更佳。下面的示例使用了 llama3 和 phi3 模型。
有关支持的模型和模型变体的完整列表，请参见 [Ollama 模型库](https://ollama.ai/library)。

## 概述

### 集成细节

|                                                                类                                                                | 包 | 本地 | 可序列化 | JS支持 | 包下载量 | 包最新版本 |
|:-------------------------------------------------------------------------------------------------------------------------------:|:---:|:---:|:-------:|:-----:|:-------:|:---------:|
| [OllamaFunctions](https://api.python.langchain.com/en/latest/llms/langchain_experimental.llms.ollama_function.OllamaFunctions.html) | [langchain-experimental](https://api.python.langchain.com/en/latest/openai_api_reference.html) | ✅ | ❌ | ❌ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-experimental?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain-experimental?style=flat-square&label=%20) |

### 模型特性

| [工具调用](/docs/how_to/tool_calling/) | [结构化输出](/docs/how_to/structured_output/) | JSON 模式 | 图像输入 | 音频输入 | 视频输入 | [令牌级流式传输](/docs/how_to/chat_streaming/) | 原生异步 | [令牌使用](/docs/how_to/chat_token_usage_tracking/) | [Logprobs](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |

## 设置

要访问 `OllamaFunctions`，您需要安装 `langchain-experimental` 集成包。请按照 [这些说明](https://github.com/jmorganca/ollama) 设置并运行本地 Ollama 实例，以及下载和提供 [支持的模型](https://ollama.com/library)。

### 凭证

目前不支持凭证。

### 安装

`OllamaFunctions` 类位于 `langchain-experimental` 包中：



```python
%pip install -qU langchain-experimental
```

## 实例化

`OllamaFunctions` 接受与 `ChatOllama` 相同的初始化参数。

为了使用工具调用，您还必须指定 `format="json"`。


```python
from langchain_experimental.llms.ollama_functions import OllamaFunctions

llm = OllamaFunctions(model="phi3")
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
AIMessage(content="J'adore programmer.", id='run-94815fcf-ae11-438a-ba3f-00819328b5cd-0')
```



```python
ai_msg.content
```



```output
"J'adore programmer."
```

## 链接

我们可以像这样使用提示模板来[链接](https://python.langchain.com/v0.2/docs/how_to/sequence/)我们的模型：


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
AIMessage(content='Programmieren ist sehr verrückt! Es freut mich, dass Sie auf Programmierung so positiv eingestellt sind.', id='run-ee99be5e-4d48-4ab6-b602-35415f0bdbde-0')
```

## 工具调用

### OllamaFunctions.bind_tools()

使用 `OllamaFunctions.bind_tools`，我们可以轻松地将 Pydantic 类、字典模式、LangChain 工具或甚至函数作为工具传递给模型。在底层，这些会被转换为工具定义模式，格式如下：


```python
from langchain_core.pydantic_v1 import BaseModel, Field


class GetWeather(BaseModel):
    """获取给定位置的当前天气"""

    location: str = Field(..., description="城市和州，例如：旧金山，加州")


llm_with_tools = llm.bind_tools([GetWeather])
```


```python
ai_msg = llm_with_tools.invoke(
    "旧金山的天气怎么样",
)
ai_msg
```



```output
AIMessage(content='', id='run-b9769435-ec6a-4cb8-8545-5a5035fc19bd-0', tool_calls=[{'name': 'GetWeather', 'args': {'location': 'San Francisco, CA'}, 'id': 'call_064c4e1cb27e4adb9e4e7ed60362ecc9'}])
```

### AIMessage.tool_calls

注意，AIMessage具有`tool_calls`属性。它包含以标准化的`ToolCall`格式表示的内容，该格式与模型提供者无关。


```python
ai_msg.tool_calls
```



```output
[{'name': 'GetWeather',
  'args': {'location': 'San Francisco, CA'},
  'id': 'call_064c4e1cb27e4adb9e4e7ed60362ecc9'}]
```


有关绑定工具和工具调用输出的更多信息，请查看[工具调用](../../how_to/function_calling.md)文档。

## API 参考

有关所有 ToolCallingLLM 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/llms/langchain_experimental.llms.ollama_functions.OllamaFunctions.html

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)