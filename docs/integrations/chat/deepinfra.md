---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/deepinfra.ipynb
---

# DeepInfra

[DeepInfra](https://deepinfra.com/?utm_source=langchain) 是一种无服务器推理服务，提供对各种 [LLMs](https://deepinfra.com/models?utm_source=langchain) 和 [嵌入模型](https://deepinfra.com/models?type=embeddings&utm_source=langchain) 的访问。此笔记本介绍了如何将 LangChain 与 DeepInfra 一起用于聊天模型。

## 设置环境 API 密钥
确保从 DeepInfra 获取您的 API 密钥。您需要 [登录](https://deepinfra.com/login?from=%2Fdash) 并获取一个新令牌。

您可以获得 1 小时的无服务器 GPU 计算免费体验，以测试不同的模型。（请参见 [这里](https://github.com/deepinfra/deepctl#deepctl)）
您可以使用 `deepctl auth token` 打印您的令牌


```python
# get a new token: https://deepinfra.com/login?from=%2Fdash

import os
from getpass import getpass

from langchain_community.chat_models import ChatDeepInfra
from langchain_core.messages import HumanMessage

DEEPINFRA_API_TOKEN = getpass()

# or pass deepinfra_api_token parameter to the ChatDeepInfra constructor
os.environ["DEEPINFRA_API_TOKEN"] = DEEPINFRA_API_TOKEN

chat = ChatDeepInfra(model="meta-llama/Llama-2-7b-chat-hf")

messages = [
    HumanMessage(
        content="Translate this sentence from English to French. I love programming."
    )
]
chat.invoke(messages)
```

## `ChatDeepInfra` 还支持异步和流式功能：


```python
from langchain_core.callbacks import StreamingStdOutCallbackHandler
```


```python
await chat.agenerate([messages])
```


```python
chat = ChatDeepInfra(
    streaming=True,
    verbose=True,
    callbacks=[StreamingStdOutCallbackHandler()],
)
chat.invoke(messages)
```

# 工具调用

DeepInfra 目前仅支持调用和异步调用工具。

有关支持工具调用的模型的完整列表，请参阅我们的 [工具调用文档](https://deepinfra.com/docs/advanced/function_calling)。

```python
import asyncio

from dotenv import find_dotenv, load_dotenv
from langchain_community.chat_models import ChatDeepInfra
from langchain_core.messages import HumanMessage
from langchain_core.pydantic_v1 import BaseModel
from langchain_core.tools import tool

model_name = "meta-llama/Meta-Llama-3-70B-Instruct"

_ = load_dotenv(find_dotenv())


# Langchain 工具
@tool
def foo(something):
    """
    当调用 foo 时
    """
    pass


# Pydantic 类
class Bar(BaseModel):
    """
    当调用 Bar 时
    """

    pass


llm = ChatDeepInfra(model=model_name)
tools = [foo, Bar]
llm_with_tools = llm.bind_tools(tools)
messages = [
    HumanMessage("请调用 Foo 和 Bar。"),
]

response = llm_with_tools.invoke(messages)
print(response.tool_calls)
# [{'name': 'foo', 'args': {'something': None}, 'id': 'call_Mi4N4wAtW89OlbizFE1aDxDj'}, {'name': 'Bar', 'args': {}, 'id': 'call_daiE0mW454j2O1KVbmET4s2r'}]


async def call_ainvoke():
    result = await llm_with_tools.ainvoke(messages)
    print(result.tool_calls)


# 异步调用
asyncio.run(call_ainvoke())
# [{'name': 'foo', 'args': {'something': None}, 'id': 'call_ZH7FetmgSot4LHcMU6CEb8tI'}, {'name': 'Bar', 'args': {}, 'id': 'call_2MQhDifAJVoijZEvH8PeFSVB'}]
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)