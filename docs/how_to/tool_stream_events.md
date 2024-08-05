---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/tool_stream_events.ipynb
---

# 如何从工具流式传输事件

:::info 前提条件

本指南假设您对以下概念有所了解：
- [LangChain 工具](/docs/concepts/#tools)
- [自定义工具](/docs/how_to/custom_tools)
- [使用流事件](/docs/how_to/streaming/#using-stream-events)
- [在自定义工具中访问 RunnableConfig](/docs/how_to/tool_configure/)

:::

如果您有调用聊天模型、检索器或其他可运行对象的工具，您可能希望访问这些可运行对象的内部事件或使用额外属性对其进行配置。本指南将向您展示如何正确手动传递参数，以便您可以使用 `astream_events()` 方法实现这一点。

:::caution 兼容性

如果您在 `python<=3.10` 中运行 `async` 代码，LangChain 无法自动传播配置，包括 `astream_events()` 所需的回调到子可运行对象。这是您可能无法从自定义可运行对象或工具中看到事件发出的常见原因。

如果您在 `python<=3.10` 中运行，您需要在异步环境中手动将 `RunnableConfig` 对象传播到子可运行对象。有关如何手动传播配置的示例，请参见下面的 `bar` RunnableLambda 的实现。

如果您在 `python>=3.11` 中运行，`RunnableConfig` 将自动传播到异步环境中的子可运行对象。然而，如果您的代码可能在较旧的 Python 版本中运行，手动传播 `RunnableConfig` 仍然是个好主意。

本指南还要求 `langchain-core>=0.2.16`。
:::

假设您有一个自定义工具，该工具调用一个链，通过提示聊天模型返回仅 10 个单词来浓缩其输入，然后反转输出。首先，以简单的方式定义它：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="model" />


```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool


@tool
async def special_summarization_tool(long_text: str) -> str:
    """一个使用高级技术总结输入文本的工具。"""
    prompt = ChatPromptTemplate.from_template(
        "您是一位专家作家。请将以下文本总结为 10 个单词或更少：\n\n{long_text}"
    )

    def reverse(x: str):
        return x[::-1]

    chain = prompt | model | StrOutputParser() | reverse
    summary = await chain.ainvoke({"long_text": long_text})
    return summary
```

直接调用工具工作得很好：


```python
LONG_TEXT = """
叙述者：
（黑屏上有文字；可以听到嗡嗡的蜜蜂声）
根据所有已知的航空法则，蜜蜂应该无法飞翔。它的翅膀太小，无法将它肥胖的小身体抬离地面。当然，蜜蜂仍然飞翔，因为蜜蜂不在乎人类认为不可能的事情。
巴里·本森：
（巴里正在挑选衬衫）
黄色，黑色。黄色，黑色。黄色，黑色。黄色，黑色。哦，黑色和黄色！让我们稍微改变一下。
珍妮特·本森：
巴里！早餐好了！
巴里：
来了！等一下。
"""

await special_summarization_tool.ainvoke({"long_text": LONG_TEXT})
```



```output
'.yad noitaudarg rof tiftuo sesoohc yrraB ;scisyhp seifed eeB'
```


但是，如果您想访问聊天模型的原始输出，而不是完整的工具，您可能会尝试使用 [`astream_events()`](/docs/how_to/streaming/#using-stream-events) 方法并查找 `on_chat_model_end` 事件。以下是发生的事情：


```python
stream = special_summarization_tool.astream_events(
    {"long_text": LONG_TEXT}, version="v2"
)

async for event in stream:
    if event["event"] == "on_chat_model_end":
        # 在 python<=3.10 中从未触发！
        print(event)
```

您会注意到（除非您在 `python>=3.11` 中运行本指南），没有从子运行中发出的聊天模型事件！

这是因为上面的示例没有将工具的配置对象传递到内部链中。要解决此问题，请重新定义您的工具，以接受一个特殊参数，类型为 `RunnableConfig`（有关更多详细信息，请参见 [本指南](/docs/how_to/tool_configure)）。在执行内部链时，您还需要将该参数传递进去：


```python
from langchain_core.runnables import RunnableConfig


@tool
async def special_summarization_tool_with_config(
    long_text: str, config: RunnableConfig
) -> str:
    """一个使用高级技术总结输入文本的工具。"""
    prompt = ChatPromptTemplate.from_template(
        "您是一位专家作家。请将以下文本总结为 10 个单词或更少：\n\n{long_text}"
    )

    def reverse(x: str):
        return x[::-1]

    chain = prompt | model | StrOutputParser() | reverse
    # 将 "config" 对象作为参数传递给任何执行的可运行对象
    summary = await chain.ainvoke({"long_text": long_text}, config=config)
    return summary
```

现在尝试与之前相同的 `astream_events()` 调用，使用您的新工具：


```python
stream = special_summarization_tool_with_config.astream_events(
    {"long_text": LONG_TEXT}, version="v2"
)

async for event in stream:
    if event["event"] == "on_chat_model_end":
        print(event)
```
```output
{'event': 'on_chat_model_end', 'data': {'output': AIMessage(content='蜜蜂挑战物理；巴里选择毕业日的服装。', response_metadata={'stop_reason': 'end_turn', 'stop_sequence': None}, id='run-d23abc80-0dce-4f74-9d7b-fb98ca4f2a9e', usage_metadata={'input_tokens': 182, 'output_tokens': 16, 'total_tokens': 198}), 'input': {'messages': [[HumanMessage(content="您是一位专家作家。请将以下文本总结为 10 个单词或更少：\n\n\n叙述者：\n（黑屏上有文字；可以听到嗡嗡的蜜蜂声）\n根据所有已知的航空法则，蜜蜂应该无法飞翔。它的翅膀太小，无法将它肥胖的小身体抬离地面。当然，蜜蜂仍然飞翔，因为蜜蜂不在乎人类认为不可能的事情。\n巴里·本森：\n（巴里正在挑选衬衫）\n黄色，黑色。黄色，黑色。黄色，黑色。黄色，黑色。哦，黑色和黄色！让我们稍微改变一下。\n珍妮特·本森：\n巴里！早餐好了！\n巴里：\n来了！等一下。\n")]]}}, 'run_id': 'd23abc80-0dce-4f74-9d7b-fb98ca4f2a9e', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['f25c41fe-8972-4893-bc40-cecf3922c1fa']}
```
太棒了！这次发出了一个事件。

对于流式传输，`astream_events()` 会自动调用链中的内部可运行对象，如果可能的话启用流式传输，因此如果您希望在聊天模型生成时流式传输标记，您只需过滤以查找 `on_chat_model_stream` 事件即可，无需其他更改：


```python
stream = special_summarization_tool_with_config.astream_events(
    {"long_text": LONG_TEXT}, version="v2"
)

async for event in stream:
    if event["event"] == "on_chat_model_stream":
        print(event)
```
```output
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='', id='run-f24ab147-0b82-4e63-810a-b12bd8d1fb42', usage_metadata={'input_tokens': 182, 'output_tokens': 0, 'total_tokens': 182})}, 'run_id': 'f24ab147-0b82-4e63-810a-b12bd8d1fb42', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['385f3612-417c-4a70-aae0-cce3a5ba6fb6']}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='蜜蜂', id='run-f24ab147-0b82-4e63-810a-b12bd8d1fb42')}, 'run_id': 'f24ab147-0b82-4e63-810a-b12bd8d1fb42', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['385f3612-417c-4a70-aae0-cce3a5ba6fb6']}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content=' 挑', id='run-f24ab147-0b82-4e63-810a-b12bd8d1fb42')}, 'run_id': 'f24ab147-0b82-4e63-810a-b12bd8d1fb42', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['385f3612-417c-4a70-aae0-cce3a5ba6fb6']}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='战物理', id='run-f24ab147-0b82-4e63-810a-b12bd8d1fb42')}, 'run_id': 'f24ab147-0b82-4e63-810a-b12bd8d1fb42', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['385f3612-417c-4a70-aae0-cce3a5ba6fb6']}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='；', id='run-f24ab147-0b82-4e63-810a-b12bd8d1fb42')}, 'run_id': 'f24ab147-0b82-4e63-810a-b12bd8d1fb42', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['385f3612-417c-4a70-aae0-cce3a5ba6fb6']}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content=' 巴里', id='run-f24ab147-0b82-4e63-810a-b12bd8d1fb42')}, 'run_id': 'f24ab147-0b82-4e63-810a-b12bd8d1fb42', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['385f3612-417c-4a70-aae0-cce3a5ba6fb6']}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content=' 选择', id='run-f24ab147-0b82-4e63-810a-b12bd8d1fb42')}, 'run_id': 'f24ab147-0b82-4e63-810a-b12bd8d1fb42', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['385f3612-417c-4a70-aae0-cce3a5ba6fb6']}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='毕业', id='run-f24ab147-0b82-4e63-810a-b12bd8d1fb42')}, 'run_id': 'f24ab147-0b82-4e63-810a-b12bd8d1fb42', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['385f3612-417c-4a70-aae0-cce3a5ba6fb6']}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='日', id='run-f24ab147-0b82-4e63-810a-b12bd8d1fb42')}, 'run_id': 'f24ab147-0b82-4e63-810a-b12bd8d1fb42', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['385f3612-417c-4a70-aae0-cce3a5ba6fb6']}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='。', id='run-f24ab147-0b82-4e63-810a-b12bd8d1fb42')}, 'run_id': 'f24ab147-0b82-4e63-810a-b12bd8d1fb42', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['385f3612-417c-4a70-aae0-cce3a5ba6fb6']}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='', response_metadata={'stop_reason': 'end_turn', 'stop_sequence': None}, id='run-f24ab147-0b82-4e63-810a-b12bd8d1fb42', usage_metadata={'input_tokens': 0, 'output_tokens': 16, 'total_tokens': 16})}, 'run_id': 'f24ab147-0b82-4e63-810a-b12bd8d1fb42', 'name': 'ChatAnthropic', 'tags': ['seq:step:2'], 'metadata': {'ls_provider': 'anthropic', 'ls_model_name': 'claude-3-5-sonnet-20240620', 'ls_model_type': 'chat', 'ls_temperature': 0.0, 'ls_max_tokens': 1024}, 'parent_ids': ['385f3612-417c-4a70-aae0-cce3a5ba6fb6']}
```

## 下一步

您现在已经了解如何在工具中流式传输事件。接下来，请查看以下指南以获取有关使用工具的更多信息：

- 将 [运行时值传递给工具](/docs/how_to/tool_runtime)
- 将 [工具结果传递回模型](/docs/how_to/tool_results_pass_to_model)
- [调度自定义回调事件](/docs/how_to/callbacks_custom_events)

您还可以查看一些工具调用的更具体用法：

- 构建 [使用工具的链和代理](/docs/how_to#tools)
- 从模型获取 [结构化输出](/docs/how_to/structured_output/)