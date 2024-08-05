---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/chat_models_universal_init.ipynb
---

# 如何在一行中初始化任何模型

许多 LLM 应用程序允许最终用户指定他们希望应用程序使用的模型提供者和模型。这需要编写一些逻辑，根据用户配置初始化不同的 ChatModels。`init_chat_model()` 辅助方法使得初始化多个不同的模型集成变得简单，无需担心导入路径和类名。

:::tip 支持的模型

请参阅 [init_chat_model()](https://api.python.langchain.com/en/latest/chat_models/langchain.chat_models.base.init_chat_model.html) API 参考以获取完整的支持集成列表。

确保已安装您希望支持的任何模型提供者的集成包。例如，您应该安装 `langchain-openai` 以初始化 OpenAI 模型。

:::

:::info 需要 ``langchain >= 0.2.8``

此功能是在 ``langchain-core == 0.2.8`` 中添加的。请确保您的包是最新的。

:::


```python
%pip install -qU langchain>=0.2.8 langchain-openai langchain-anthropic langchain-google-vertexai
```

## 基本用法


```python
from langchain.chat_models import init_chat_model

# Returns a langchain_openai.ChatOpenAI instance.
gpt_4o = init_chat_model("gpt-4o", model_provider="openai", temperature=0)
# Returns a langchain_anthropic.ChatAnthropic instance.
claude_opus = init_chat_model(
    "claude-3-opus-20240229", model_provider="anthropic", temperature=0
)
# Returns a langchain_google_vertexai.ChatVertexAI instance.
gemini_15 = init_chat_model(
    "gemini-1.5-pro", model_provider="google_vertexai", temperature=0
)

# Since all model integrations implement the ChatModel interface, you can use them in the same way.
print("GPT-4o: " + gpt_4o.invoke("what's your name").content + "\n")
print("Claude Opus: " + claude_opus.invoke("what's your name").content + "\n")
print("Gemini 1.5: " + gemini_15.invoke("what's your name").content + "\n")
```
```output
GPT-4o: I'm an AI created by OpenAI, and I don't have a personal name. You can call me Assistant! How can I help you today?

Claude Opus: My name is Claude. It's nice to meet you!

Gemini 1.5: I am a large language model, trained by Google. I do not have a name.
```

## 推断模型提供者

对于常见和特定的模型名称，`init_chat_model()` 将尝试推断模型提供者。有关推断行为的完整列表，请参见 [API 参考](https://api.python.langchain.com/en/latest/chat_models/langchain.chat_models.base.init_chat_model.html)。例如，任何以 `gpt-3...` 或 `gpt-4...` 开头的模型将被推断为使用模型提供者 `openai`。

```python
gpt_4o = init_chat_model("gpt-4o", temperature=0)
claude_opus = init_chat_model("claude-3-opus-20240229", temperature=0)
gemini_15 = init_chat_model("gemini-1.5-pro", temperature=0)
```

## 创建可配置模型

您还可以通过指定 `configurable_fields` 来创建一个运行时可配置的模型。如果您不指定 `model` 值，则 "model" 和 "model_provider" 默认是可配置的。

```python
configurable_model = init_chat_model(temperature=0)

configurable_model.invoke(
    "what's your name", config={"configurable": {"model": "gpt-4o"}}
)
```



```output
AIMessage(content="I'm an AI language model created by OpenAI, and I don't have a personal name. You can call me Assistant or any other name you prefer! How can I assist you today?", response_metadata={'token_usage': {'completion_tokens': 37, 'prompt_tokens': 11, 'total_tokens': 48}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_d576307f90', 'finish_reason': 'stop', 'logprobs': None}, id='run-5428ab5c-b5c0-46de-9946-5d4ca40dbdc8-0', usage_metadata={'input_tokens': 11, 'output_tokens': 37, 'total_tokens': 48})
```



```python
configurable_model.invoke(
    "what's your name", config={"configurable": {"model": "claude-3-5-sonnet-20240620"}}
)
```



```output
AIMessage(content="My name is Claude. It's nice to meet you!", response_metadata={'id': 'msg_012XvotUJ3kGLXJUWKBVxJUi', 'model': 'claude-3-5-sonnet-20240620', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 11, 'output_tokens': 15}}, id='run-1ad1eefe-f1c6-4244-8bc6-90e2cb7ee554-0', usage_metadata={'input_tokens': 11, 'output_tokens': 15, 'total_tokens': 26})
```

### 可配置模型与默认值

我们可以创建一个具有默认模型值的可配置模型，指定哪些参数是可配置的，并为可配置参数添加前缀：

```python
first_llm = init_chat_model(
    model="gpt-4o",
    temperature=0,
    configurable_fields=("model", "model_provider", "temperature", "max_tokens"),
    config_prefix="first",  # 当您有多个模型的链时，这很有用
)

first_llm.invoke("what's your name")
```

```output
AIMessage(content="I'm an AI language model created by OpenAI, and I don't have a personal name. You can call me Assistant or any other name you prefer! How can I assist you today?", response_metadata={'token_usage': {'completion_tokens': 37, 'prompt_tokens': 11, 'total_tokens': 48}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_ce0793330f', 'finish_reason': 'stop', 'logprobs': None}, id='run-3923e328-7715-4cd6-b215-98e4b6bf7c9d-0', usage_metadata={'input_tokens': 11, 'output_tokens': 37, 'total_tokens': 48})
```

```python
first_llm.invoke(
    "what's your name",
    config={
        "configurable": {
            "first_model": "claude-3-5-sonnet-20240620",
            "first_temperature": 0.5,
            "first_max_tokens": 100,
        }
    },
)
```

```output
AIMessage(content="My name is Claude. It's nice to meet you!", response_metadata={'id': 'msg_01RyYR64DoMPNCfHeNnroMXm', 'model': 'claude-3-5-sonnet-20240620', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 11, 'output_tokens': 15}}, id='run-22446159-3723-43e6-88df-b84797e7751d-0', usage_metadata={'input_tokens': 11, 'output_tokens': 15, 'total_tokens': 26})
```

### 以声明方式使用可配置模型

我们可以在可配置模型上调用声明性操作，如 `bind_tools`、`with_structured_output`、`with_configurable` 等，并以与常规实例化的聊天模型对象相同的方式链接可配置模型。

```python
from langchain_core.pydantic_v1 import BaseModel, Field


class GetWeather(BaseModel):
    """获取指定位置的当前天气"""

    location: str = Field(..., description="城市和州，例如：旧金山，加州")


class GetPopulation(BaseModel):
    """获取指定位置的当前人口"""

    location: str = Field(..., description="城市和州，例如：旧金山，加州")


llm = init_chat_model(temperature=0)
llm_with_tools = llm.bind_tools([GetWeather, GetPopulation])

llm_with_tools.invoke(
    "2024年哪个更大，洛杉矶还是纽约", config={"configurable": {"model": "gpt-4o"}}
).tool_calls
```



```output
[{'name': 'GetPopulation',
  'args': {'location': 'Los Angeles, CA'},
  'id': 'call_sYT3PFMufHGWJD32Hi2CTNUP'},
 {'name': 'GetPopulation',
  'args': {'location': 'New York, NY'},
  'id': 'call_j1qjhxRnD3ffQmRyqjlI1Lnk'}]
```



```python
llm_with_tools.invoke(
    "2024年哪个更大，洛杉矶还是纽约",
    config={"configurable": {"model": "claude-3-5-sonnet-20240620"}},
).tool_calls
```



```output
[{'name': 'GetPopulation',
  'args': {'location': 'Los Angeles, CA'},
  'id': 'toolu_01CxEHxKtVbLBrvzFS7GQ5xR'},
 {'name': 'GetPopulation',
  'args': {'location': 'New York City, NY'},
  'id': 'toolu_013A79qt5toWSsKunFBDZd5S'}]
```