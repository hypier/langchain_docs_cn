---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/custom_chat_model.ipynb
---

# 如何创建自定义聊天模型类

:::info 先决条件

本指南假设您对以下概念有一定了解：
- [聊天模型](/docs/concepts/#chat-models)

:::

在本指南中，我们将学习如何使用 LangChain 抽象创建自定义聊天模型。

使用标准 [`BaseChatModel`](https://api.python.langchain.com/en/latest/language_models/langchain_core.language_models.chat_models.BaseChatModel.html) 接口包装您的 LLM，可以让您在现有的 LangChain 程序中以最小的代码修改使用您的 LLM！

作为额外好处，您的 LLM 将自动成为 LangChain `Runnable`，并将受益于一些开箱即用的优化（例如，通过线程池批处理）、异步支持、`astream_events` API 等。

## 输入和输出

首先，我们需要讨论一下 **消息**，它们是聊天模型的输入和输出。

### 消息

聊天模型将消息作为输入，并返回消息作为输出。

LangChain 有几种 [内置消息类型](/docs/concepts/#message-types)：

| 消息类型              | 描述                                                                                          |
|-----------------------|-------------------------------------------------------------------------------------------------|
| `SystemMessage`       | 用于初始化 AI 行为，通常作为输入消息序列中的第一个传入。                                         |
| `HumanMessage`        | 代表与聊天模型互动的人的消息。                                                                |
| `AIMessage`           | 代表来自聊天模型的消息。这可以是文本或请求调用工具。                                          |
| `FunctionMessage` / `ToolMessage` | 用于将工具调用的结果传回模型的消息。                                      |
| `AIMessageChunk` / `HumanMessageChunk` / ... | 每种消息类型的块变体。 |


::: {.callout-note}
`ToolMessage` 和 `FunctionMessage` 紧密遵循 OpenAI 的 `function` 和 `tool` 角色。

这是一个快速发展的领域，随着更多模型添加功能调用能力，预计该架构将会有新增内容。
:::


```python
from langchain_core.messages import (
    AIMessage,
    BaseMessage,
    FunctionMessage,
    HumanMessage,
    SystemMessage,
    ToolMessage,
)
```

### 流式变体

所有聊天消息都有一个流式变体，其名称中包含 `Chunk`。

```python
from langchain_core.messages import (
    AIMessageChunk,
    FunctionMessageChunk,
    HumanMessageChunk,
    SystemMessageChunk,
    ToolMessageChunk,
)
```

这些块在从聊天模型流式输出时使用，它们都定义了一个可加属性！

```python
AIMessageChunk(content="Hello") + AIMessageChunk(content=" World!")
```



```output
AIMessageChunk(content='Hello World!')
```

## 基础聊天模型

让我们实现一个聊天模型，它会回显提示中最后一条消息的前 `n` 个字符！

为此，我们将继承 `BaseChatModel`，并需要实现以下内容：

| 方法/属性                           | 描述                                                             | 必需/可选          |
|------------------------------------|-------------------------------------------------------------------|--------------------|
| `_generate`                        | 用于从提示生成聊天结果                                           | 必需               |
| `_llm_type` (属性)                 | 用于唯一识别模型类型。用于日志记录。                             | 必需               |
| `_identifying_params` (属性)       | 表示模型参数化以便追踪目的。                                     | 可选               |
| `_stream`                          | 用于实现流式传输。                                               | 可选               |
| `_agenerate`                       | 用于实现原生异步方法。                                           | 可选               |
| `_astream`                         | 用于实现 `_stream` 的异步版本。                                  | 可选               |


:::tip
`_astream` 实现使用 `run_in_executor` 在单独的线程中启动同步的 `_stream`，如果已实现 `_stream`，否则回退使用 `_agenerate`。

如果您想重用 `_stream` 实现，可以使用这个技巧，但如果您能够实现原生异步的代码，那将是更好的解决方案，因为该代码的运行开销更小。
:::

### 实现


```python
from typing import Any, AsyncIterator, Dict, Iterator, List, Optional

from langchain_core.callbacks import (
    AsyncCallbackManagerForLLMRun,
    CallbackManagerForLLMRun,
)
from langchain_core.language_models import BaseChatModel, SimpleChatModel
from langchain_core.messages import AIMessageChunk, BaseMessage, HumanMessage
from langchain_core.outputs import ChatGeneration, ChatGenerationChunk, ChatResult
from langchain_core.runnables import run_in_executor


class CustomChatModelAdvanced(BaseChatModel):
    """一个自定义聊天模型，回显输入的前 `n` 个字符。

    在向 LangChain 提交实现时，仔细记录模型，包括初始化参数，
    包括如何初始化模型的示例，并包含任何相关的底层模型文档或 API 的链接。

    示例：

        .. code-block:: python

            model = CustomChatModel(n=2)
            result = model.invoke([HumanMessage(content="hello")])
            result = model.batch([[HumanMessage(content="hello")],
                                 [HumanMessage(content="world")]])
    """

    model_name: str
    """模型的名称"""
    n: int
    """要回显的提示最后一条消息的字符数。"""

    def _generate(
        self,
        messages: List[BaseMessage],
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> ChatResult:
        """重写 _generate 方法以实现聊天模型逻辑。

        这可以是对 API 的调用、对本地模型的调用或任何其他
        生成对输入提示的响应的实现。

        参数：
            messages: 由消息列表组成的提示。
            stop: 模型应停止生成的字符串列表。
                  如果由于停止标记而停止生成，则停止标记本身
                  应该作为输出的一部分包含。这在目前的模型中没有强制执行，
                  但遵循这一良好实践可以使后续解析模型输出
                  更加容易，并理解生成停止的原因。
            run_manager: 带有回调的 LLM 运行管理器。
        """
        # 用实际逻辑替换此处，以从消息列表生成响应。
        last_message = messages[-1]
        tokens = last_message.content[: self.n]
        message = AIMessage(
            content=tokens,
            additional_kwargs={},  # 用于添加额外的负载（例如，函数调用请求）
            response_metadata={  # 用于响应元数据
                "time_in_seconds": 3,
            },
        )
        ##

        generation = ChatGeneration(message=message)
        return ChatResult(generations=[generation])

    def _stream(
        self,
        messages: List[BaseMessage],
        stop: Optional[List[str]] = None,
        run_manager: Optional[CallbackManagerForLLMRun] = None,
        **kwargs: Any,
    ) -> Iterator[ChatGenerationChunk]:
        """流式输出模型的结果。

        如果模型可以以流式方式生成输出，则应实现此方法。
        如果模型不支持流式处理，则不应实现它。在这种情况下，流式请求将自动
        由 _generate 方法处理。

        参数：
            messages: 由消息列表组成的提示。
            stop: 模型应停止生成的字符串列表。
                  如果由于停止标记而停止生成，则停止标记本身
                  应该作为输出的一部分包含。这在目前的模型中没有强制执行，
                  但遵循这一良好实践可以使后续解析模型输出
                  更加容易，并理解生成停止的原因。
            run_manager: 带有回调的 LLM 运行管理器。
        """
        last_message = messages[-1]
        tokens = last_message.content[: self.n]

        for token in tokens:
            chunk = ChatGenerationChunk(message=AIMessageChunk(content=token))

            if run_manager:
                # 这是在较新版本的 LangChain 中是可选的
                # on_llm_new_token 将自动被调用
                run_manager.on_llm_new_token(token, chunk=chunk)

            yield chunk

        # 让我们添加一些其他信息（例如，响应元数据）
        chunk = ChatGenerationChunk(
            message=AIMessageChunk(content="", response_metadata={"time_in_sec": 3})
        )
        if run_manager:
            # 这是在较新版本的 LangChain 中是可选的
            # on_llm_new_token 将自动被调用
            run_manager.on_llm_new_token(token, chunk=chunk)
        yield chunk

    @property
    def _llm_type(self) -> str:
        """获取此聊天模型使用的语言模型类型。"""
        return "echoing-chat-model-advanced"

    @property
    def _identifying_params(self) -> Dict[str, Any]:
        """返回识别参数的字典。

        此信息由 LangChain 回调系统使用，用于跟踪目的，使监控 LLM 成为可能。
        """
        return {
            # 模型名称允许用户在 LLM 监控应用程序中指定自定义令牌计数
            # 规则（例如，在 LangSmith 中，用户可以为其模型提供每个令牌的定价并监控
            # 给定 LLM 的成本。）
            "model_name": self.model_name,
        }
```

### 让我们测试一下 🧪

聊天模型将实现 LangChain 的标准 `Runnable` 接口，许多 LangChain 抽象都支持该接口！


```python
model = CustomChatModelAdvanced(n=3, model_name="my_custom_model")

model.invoke(
    [
        HumanMessage(content="hello!"),
        AIMessage(content="Hi there human!"),
        HumanMessage(content="Meow!"),
    ]
)
```



```output
AIMessage(content='Meo', response_metadata={'time_in_seconds': 3}, id='run-ddb42bd6-4fdd-4bd2-8be5-e11b67d3ac29-0')
```



```python
model.invoke("hello")
```



```output
AIMessage(content='hel', response_metadata={'time_in_seconds': 3}, id='run-4d3cc912-44aa-454b-977b-ca02be06c12e-0')
```



```python
model.batch(["hello", "goodbye"])
```



```output
[AIMessage(content='hel', response_metadata={'time_in_seconds': 3}, id='run-9620e228-1912-4582-8aa1-176813afec49-0'),
 AIMessage(content='goo', response_metadata={'time_in_seconds': 3}, id='run-1ce8cdf8-6f75-448e-82f7-1bb4a121df93-0')]
```



```python
for chunk in model.stream("cat"):
    print(chunk.content, end="|")
```
```output
c|a|t||
```
请查看模型中 `_astream` 的实现！如果您没有实现它，则不会有输出流。！


```python
async for chunk in model.astream("cat"):
    print(chunk.content, end="|")
```
```output
c|a|t||
```
让我们尝试使用 astream 事件 API，这也将帮助双重检查所有回调是否已实现！


```python
async for event in model.astream_events("cat", version="v1"):
    print(event)
```
```output
{'event': 'on_chat_model_start', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'name': 'CustomChatModelAdvanced', 'tags': [], 'metadata': {}, 'data': {'input': 'cat'}}
{'event': 'on_chat_model_stream', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'tags': [], 'metadata': {}, 'name': 'CustomChatModelAdvanced', 'data': {'chunk': AIMessageChunk(content='c', id='run-125a2a16-b9cd-40de-aa08-8aa9180b07d0')}}
{'event': 'on_chat_model_stream', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'tags': [], 'metadata': {}, 'name': 'CustomChatModelAdvanced', 'data': {'chunk': AIMessageChunk(content='a', id='run-125a2a16-b9cd-40de-aa08-8aa9180b07d0')}}
{'event': 'on_chat_model_stream', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'tags': [], 'metadata': {}, 'name': 'CustomChatModelAdvanced', 'data': {'chunk': AIMessageChunk(content='t', id='run-125a2a16-b9cd-40de-aa08-8aa9180b07d0')}}
{'event': 'on_chat_model_stream', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'tags': [], 'metadata': {}, 'name': 'CustomChatModelAdvanced', 'data': {'chunk': AIMessageChunk(content='', response_metadata={'time_in_sec': 3}, id='run-125a2a16-b9cd-40de-aa08-8aa9180b07d0')}}
{'event': 'on_chat_model_end', 'name': 'CustomChatModelAdvanced', 'run_id': '125a2a16-b9cd-40de-aa08-8aa9180b07d0', 'tags': [], 'metadata': {}, 'data': {'output': AIMessageChunk(content='cat', response_metadata={'time_in_sec': 3}, id='run-125a2a16-b9cd-40de-aa08-8aa9180b07d0')}}
``````output
/home/eugene/src/langchain/libs/core/langchain_core/_api/beta_decorator.py:87: LangChainBetaWarning: 此 API 处于测试阶段，未来可能会更改。
  warn_beta(
```

## 贡献

我们非常感谢所有聊天模型集成的贡献。

以下是一个清单，以帮助确保您的贡献被添加到 LangChain 中：

文档：

* 模型包含所有初始化参数的文档字符串，因为这些将在 [APIReference](https://api.python.langchain.com/en/stable/langchain_api_reference.html) 中显示。
* 如果模型由服务提供支持，则模型的类文档字符串包含指向模型 API 的链接。

测试：

* [ ] 为重写的方法添加单元测试或集成测试。如果您重写了相应的代码，请验证 `invoke`、`ainvoke`、`batch`、`stream` 是否正常工作。

流式传输（如果您正在实现）：

* [ ] 实现 _stream 方法以使流式传输正常工作。

停止标记行为：

* [ ] 应尊重停止标记。
* [ ] 停止标记应作为响应的一部分包含在内。

秘密 API 密钥：

* [ ] 如果您的模型连接到 API，它可能会在初始化时接受 API 密钥。使用 Pydantic 的 `SecretStr` 类型来处理秘密，以便在有人打印模型时不会意外打印出来。

识别参数：

* [ ] 在识别参数中包含 `model_name`。

优化：

考虑提供原生异步支持，以减少模型的开销！

* [ ] 提供 `_agenerate` 的原生异步支持（用于 `ainvoke`）。
* [ ] 提供 `_astream` 的原生异步支持（用于 `astream`）。

## 下一步

您现在已经学习了如何创建自己的自定义聊天模型。

接下来，请查看本节中其他关于聊天模型的操作指南，例如 [如何让模型返回结构化输出](/docs/how_to/structured_output) 或 [如何跟踪聊天模型的令牌使用情况](/docs/how_to/chat_token_usage_tracking)。