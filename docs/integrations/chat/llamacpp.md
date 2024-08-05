---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/llamacpp.ipynb
---

# ChatLlamaCpp

本笔记本提供了与 [llama cpp python](https://github.com/abetlen/llama-cpp-python) 集成的聊天模型入门的快速概述。

## 概述

### 集成细节
| 类别 | 包 | 本地 | 可序列化 | JS 支持 |
| :--- | :--- | :---: | :---: |  :---: |
| [ChatLlamaCpp](https://api.python.langchain.com/en/latest/chat_models/langchain_community.chat_models.llamacpp.ChatLlamaCpp.html) | [langchain-community](https://api.python.langchain.com/en/latest/community_api_reference.html) | ✅ | ❌ | ❌ |

### 模型特性
| [工具调用](/docs/how_to/tool_calling) | [结构化输出](/docs/how_to/structured_output/) | JSON 模式 | 图像输入 | 音频输入 | 视频输入 | [令牌级流式传输](/docs/how_to/chat_streaming/) | 原生异步 | [令牌使用](/docs/how_to/chat_token_usage_tracking/) | [Logprobs](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ |

## 设置

要开始使用下面展示的**所有**功能，我们建议使用一个经过工具调用微调的模型。

我们将使用 [
Hermes-2-Pro-Llama-3-8B-GGUF](https://huggingface.co/NousResearch/Hermes-2-Pro-Llama-3-8B-GGUF) 来自NousResearch。

> Hermes 2 Pro是Nous Hermes 2的升级版本，包含更新和清理后的OpenHermes 2.5数据集，以及新引入的内部开发的函数调用和JSON模式数据集。这个新版本的Hermes保持了其出色的通用任务和对话能力——同时在函数调用方面也表现出色。

请查看我们的本地模型指南以深入了解：

* [本地运行LLMs](https://python.langchain.com/v0.1/docs/guides/development/local_llms/)
* [将本地模型与RAG结合使用](https://python.langchain.com/v0.1/docs/use_cases/question_answering/local_retrieval_qa/)

### 安装

LangChain LlamaCpp 集成位于 `langchain-community` 和 `llama-cpp-python` 包中：

```python
%pip install -qU langchain-community llama-cpp-python
```

## 实例化

现在我们可以实例化我们的模型对象并生成聊天完成内容：


```python
# Path to your model weights
local_model = "local/path/to/Hermes-2-Pro-Llama-3-8B-Q8_0.gguf"
```


```python
import multiprocessing

from langchain_community.chat_models import ChatLlamaCpp

llm = ChatLlamaCpp(
    temperature=0.5,
    model_path=local_model,
    n_ctx=10000,
    n_gpu_layers=8,
    n_batch=300,  # Should be between 1 and n_ctx, consider the amount of VRAM in your GPU.
    max_tokens=512,
    n_threads=multiprocessing.cpu_count() - 1,
    repeat_penalty=1.5,
    top_p=0.5,
    verbose=True,
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


```python
print(ai_msg.content)
```
```output
J'aime programmer. (在法国，“编程”通常用于其原始意义，即安排或组织活动。)

如果你指的是计算机编程：
Je suis amoureux de la programmation informatique.

（你也可以简单地说“programmation”，这在上下文中会被理解为两种含义。）
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

## 工具调用

首先，它的工作方式与 OpenAI 函数调用大致相同。

OpenAI 有一个 [工具调用](https://platform.openai.com/docs/guides/function-calling)（在这里我们将“工具调用”和“函数调用”互换使用）API，允许您描述工具及其参数，并让模型返回一个包含要调用的工具及其输入的 JSON 对象。工具调用对于构建使用工具的链和代理非常有用，并且通常可以从模型中获取结构化输出。

通过 `ChatLlamaCpp.bind_tools`，我们可以轻松地将 Pydantic 类、字典模式、LangChain 工具或甚至函数作为工具传递给模型。在底层，这些被转换为 OpenAI 工具模式，格式如下：
```
{
    "name": "...",
    "description": "...",
    "parameters": {...}  # JSONSchema
}
```
并在每次模型调用中传递。

然而，它无法自动触发一个函数/工具，我们需要通过指定“工具选择”参数来强制执行。该参数通常按照以下格式进行格式化。

```{"type": "function", "function": {"name": <<tool_name>>}}.```


```python
from langchain.tools import tool
from langchain_core.pydantic_v1 import BaseModel, Field


class WeatherInput(BaseModel):
    location: str = Field(description="城市和州，例如：旧金山，加州")
    unit: str = Field(enum=["celsius", "fahrenheit"])


@tool("get_current_weather", args_schema=WeatherInput)
def get_weather(location: str, unit: str):
    """获取给定位置的当前天气"""
    return f"现在{location}的天气是22 {unit}"


llm_with_tools = llm.bind_tools(
    tools=[get_weather],
    tool_choice={"type": "function", "function": {"name": "get_current_weather"}},
)
```


```python
ai_msg = llm_with_tools.invoke(
    "HCMC的天气怎么样，单位为摄氏度",
)
```


```python
ai_msg.tool_calls
```



```output
[{'name': 'get_current_weather',
  'args': {'location': '胡志明市', 'unit': 'celsius'},
  'id': 'call__0_get_current_weather_cmpl-394d9943-0a1f-425b-8139-d2826c1431f2'}]
```



```python
class MagicFunctionInput(BaseModel):
    magic_function_input: int = Field(description="魔法函数的输入值")


@tool("get_magic_function", args_schema=MagicFunctionInput)
def magic_function(magic_function_input: int):
    """获取给定输入的魔法函数值。"""
    return magic_function_input + 2


llm_with_tools = llm.bind_tools(
    tools=[magic_function],
    tool_choice={"type": "function", "function": {"name": "get_magic_function"}},
)

ai_msg = llm_with_tools.invoke(
    "3的魔法函数是什么？",
)

ai_msg
```


```python
ai_msg.tool_calls
```



```output
[{'name': 'get_magic_function',
  'args': {'magic_function_input': 3},
  'id': 'call__0_get_magic_function_cmpl-cd83a994-b820-4428-957c-48076c68335a'}]
```

# 结构化输出


```python
from langchain_core.pydantic_v1 import BaseModel
from langchain_core.utils.function_calling import convert_to_openai_tool


class Joke(BaseModel):
    """一个笑话的开场和结尾。"""

    setup: str
    punchline: str


dict_schema = convert_to_openai_tool(Joke)
structured_llm = llm.with_structured_output(dict_schema)
result = structured_llm.invoke("告诉我一个关于鸟的笑话")
result
```


```python
result
```



```output
{'setup': '- 为什么鸡要穿越游乐场?',
 'punchline': '\n\n- 为了到达另一边的华丽鸟笼!'}
```

# 流式传输



```python
for chunk in llm.stream("what is 25x5"):
    print(chunk.content, end="\n", flush=True)
```

## API 参考

有关所有 ChatLlamaCpp 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/chat_models/langchain_community.chat_models.llamacpp.ChatLlamaCpp.html

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)