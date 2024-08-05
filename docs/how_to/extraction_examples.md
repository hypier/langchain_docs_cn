---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/extraction_examples.ipynb
---

# 如何在提取时使用参考示例

通过向 LLM 提供参考示例，提取的质量通常可以得到改善。

数据提取旨在生成文本及其他非结构化或半结构化格式中所发现信息的结构化表示。此上下文中通常使用 [Tool-calling](/docs/concepts#functiontool-calling) LLM 特性。本指南演示如何构建工具调用的少量示例，以帮助引导提取及类似应用的行为。

:::tip
虽然本指南专注于如何使用工具调用模型的示例，但该技术通常适用，并且也适用于基于 JSON 或提示的技术。
:::

LangChain 在包含工具调用的 LLM 消息上实现了 [tool-call 属性](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessage.html#langchain_core.messages.ai.AIMessage.tool_calls)。有关更多详细信息，请查看我们的 [工具调用指南](/docs/how_to/tool_calling)。为了构建数据提取的参考示例，我们构建一个包含以下序列的聊天历史：

- [HumanMessage](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.human.HumanMessage.html) 包含示例输入；
- [AIMessage](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessage.html) 包含示例工具调用；
- [ToolMessage](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.tool.ToolMessage.html) 包含示例工具输出。

LangChain 采用这种约定在不同 LLM 模型提供者之间结构化工具调用的对话。

首先，我们构建一个包含这些消息占位符的提示模板：

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

# 定义一个自定义提示以提供指示和任何额外上下文。
# 1) 您可以在提示模板中添加示例以提高提取质量
# 2) 引入额外参数以考虑上下文（例如，包含提取文本的文档的元数据。）
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are an expert extraction algorithm. "
            "Only extract relevant information from the text. "
            "If you do not know the value of an attribute asked "
            "to extract, return null for the attribute's value.",
        ),
        # ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
        MessagesPlaceholder("examples"),  # <-- 示例！
        # ↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑
        ("human", "{text}"),
    ]
)
```

测试模板：

```python
from langchain_core.messages import (
    HumanMessage,
)

prompt.invoke(
    {"text": "this is some text", "examples": [HumanMessage(content="testing 1 2 3")]}
)
```

```output
ChatPromptValue(messages=[SystemMessage(content="You are an expert extraction algorithm. Only extract relevant information from the text. If you do not know the value of an attribute asked to extract, return null for the attribute's value."), HumanMessage(content='testing 1 2 3'), HumanMessage(content='this is some text')])
```

## 定义模式

让我们重用来自 [提取教程](/docs/tutorials/extraction) 的人员模式。

```python
from typing import List, Optional

from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI


class Person(BaseModel):
    """关于一个人的信息。"""

    # ^ 实体 Person 的文档字符串。
    # 该文档字符串作为模式 Person 的描述发送给 LLM，
    # 并且可以帮助改善提取结果。

    # 注意：
    # 1. 每个字段都是 `optional` -- 这允许模型拒绝提取它！
    # 2. 每个字段都有一个 `description` -- 这个描述被 LLM 使用。
    # 有一个好的描述可以帮助改善提取结果。
    name: Optional[str] = Field(..., description="这个人的名字")
    hair_color: Optional[str] = Field(
        ..., description="如果已知，这个人的头发颜色"
    )
    height_in_meters: Optional[str] = Field(..., description="以米为单位的高度")


class Data(BaseModel):
    """关于人们的提取数据。"""

    # 创建一个模型，以便我们可以提取多个实体。
    people: List[Person]
```

## 定义参考示例

示例可以定义为输入-输出对的列表。

每个示例包含一个示例 `input` 文本和一个示例 `output`，显示应从文本中提取的内容。

:::important
这部分内容比较细节，可以选择跳过。

示例的格式需要与所使用的 API 匹配（例如，工具调用或 JSON 模式等）。

在这里，格式化的示例将匹配工具调用 API 的预期格式，因为我们正在使用这个。
:::


```python
import uuid
from typing import Dict, List, TypedDict

from langchain_core.messages import (
    AIMessage,
    BaseMessage,
    HumanMessage,
    SystemMessage,
    ToolMessage,
)
from langchain_core.pydantic_v1 import BaseModel, Field


class Example(TypedDict):
    """一个示例的表示，由文本输入和预期的工具调用组成。

    对于提取，工具调用表示为 pydantic 模型的实例。
    """

    input: str  # 这是示例文本
    tool_calls: List[BaseModel]  # 应该提取的 pydantic 模型实例


def tool_example_to_messages(example: Example) -> List[BaseMessage]:
    """将示例转换为可以输入 LLM 的消息列表。

    这段代码是一个适配器，将我们的示例转换为可以输入聊天模型的消息列表。

    每个示例的消息列表对应于：

    1) HumanMessage: 包含应提取内容的文本。
    2) AIMessage: 包含从模型提取的信息
    3) ToolMessage: 向模型确认模型正确请求了工具。

    ToolMessage 是必需的，因为某些聊天模型是针对代理进行超优化的
    而不是针对提取用例。
    """
    messages: List[BaseMessage] = [HumanMessage(content=example["input"])]
    tool_calls = []
    for tool_call in example["tool_calls"]:
        tool_calls.append(
            {
                "id": str(uuid.uuid4()),
                "args": tool_call.dict(),
                # 目前函数的名称对应于
                # pydantic 模型的名称
                # 这在 API 中是隐式的，
                # 将随着时间的推移而改进。
                "name": tool_call.__class__.__name__,
            },
        )
    messages.append(AIMessage(content="", tool_calls=tool_calls))
    tool_outputs = example.get("tool_outputs") or [
        "您已正确调用此工具。"
    ] * len(tool_calls)
    for output, tool_call in zip(tool_outputs, tool_calls):
        messages.append(ToolMessage(content=output, tool_call_id=tool_call["id"]))
    return messages
```

接下来，让我们定义示例，然后将其转换为消息格式。


```python
examples = [
    (
        "海洋浩瀚而蔚蓝。深度超过 20,000 英尺。里面有很多鱼。",
        Data(people=[]),
    ),
    (
        "菲奥娜从法国远道而来，前往西班牙。",
        Data(people=[Person(name="Fiona", height_in_meters=None, hair_color=None)]),
    ),
]


messages = []

for text, tool_call in examples:
    messages.extend(
        tool_example_to_messages({"input": text, "tool_calls": [tool_call]})
    )
```

让我们测试一下提示


```python
example_prompt = prompt.invoke({"text": "这是一段文本", "examples": messages})

for message in example_prompt.messages:
    print(f"{message.type}: {message}")
```
```output
system: content="您是一个专家级的提取算法。只提取文本中的相关信息。如果您不知道要提取的属性的值，请返回该属性值的 null。"
human: content="海洋浩瀚而蔚蓝。深度超过 20,000 英尺。里面有很多鱼。"
ai: content='' tool_calls=[{'name': 'Person', 'args': {'name': None, 'hair_color': None, 'height_in_meters': None}, 'id': 'b843ba77-4c9c-48ef-92a4-54e534f24521'}]
tool: content='您已正确调用此工具。' tool_call_id='b843ba77-4c9c-48ef-92a4-54e534f24521'
human: content='菲奥娜从法国远道而来，前往西班牙。'
ai: content='' tool_calls=[{'name': 'Person', 'args': {'name': 'Fiona', 'hair_color': None, 'height_in_meters': None}, 'id': '46f00d6b-50e5-4482-9406-b07bb10340f6'}]
tool: content='您已正确调用此工具。' tool_call_id='46f00d6b-50e5-4482-9406-b07bb10340f6'
human: content='这是一段文本'
```

## 创建提取器

让我们选择一个 LLM。因为我们正在使用工具调用，所以我们需要一个支持工具调用功能的模型。请参见 [此表](/docs/integrations/chat) 以获取可用的 LLM。

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs
  customVarName="llm"
  openaiParams={`model="gpt-4-0125-preview", temperature=0`}
/>

按照 [提取教程](/docs/tutorials/extraction)，我们使用 `.with_structured_output` 方法根据所需的架构来结构化模型输出：


```python
runnable = prompt | llm.with_structured_output(
    schema=Data,
    method="function_calling",
    include_raw=False,
)
```

## 没有示例 😿

请注意，即使是能力强大的模型也可能在**非常简单**的测试用例中失败！


```python
for _ in range(5):
    text = "The solar system is large, but earth has only 1 moon."
    print(runnable.invoke({"text": text, "examples": []}))
```
```output
people=[Person(name='earth', hair_color='null', height_in_meters='null')]
people=[Person(name='earth', hair_color='null', height_in_meters='null')]
people=[]
people=[Person(name='earth', hair_color='null', height_in_meters='null')]
people=[]
```

## 带示例 😻

参考示例有助于修复故障！


```python
for _ in range(5):
    text = "The solar system is large, but earth has only 1 moon."
    print(runnable.invoke({"text": text, "examples": messages}))
```
```output
people=[]
people=[]
people=[]
people=[]
people=[]
```
请注意，我们可以在 [Langsmith trace](https://smith.langchain.com/public/4c436bc2-a1ce-440b-82f5-093947542e40/r) 中看到几次示例作为工具调用。

并且我们在一个正样本上保持性能：


```python
runnable.invoke(
    {
        "text": "My name is Harrison. My hair is black.",
        "examples": messages,
    }
)
```



```output
Data(people=[Person(name='Harrison', hair_color='black', height_in_meters=None)])
```