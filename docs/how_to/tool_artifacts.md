---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/tool_artifacts.ipynb
---

# 如何从工具返回工件

:::info 先决条件
本指南假设您熟悉以下概念：

- [ToolMessage](/docs/concepts/#toolmessage)
- [Tools](/docs/concepts/#tools)
- [Function/tool calling](/docs/concepts/#functiontool-calling)

:::

工具是可以被模型调用的实用程序，其输出旨在反馈给模型。然而，有时我们希望将工具执行的工件提供给链或代理中的下游组件，但又不想将其暴露给模型本身。例如，如果一个工具返回一个自定义对象、数据框或图像，我们可能希望将一些关于该输出的元数据传递给模型，而不传递实际输出。同时，我们可能希望能够在其他地方访问这个完整的输出，例如在下游工具中。

Tool 和 [ToolMessage](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.tool.ToolMessage.html) 接口使我们能够区分工具输出中用于模型的部分（这是 ToolMessage.content）和用于模型外部的部分（ToolMessage.artifact）。

:::info 需要 ``langchain-core >= 0.2.19``

此功能是在 ``langchain-core == 0.2.19`` 中添加的。请确保您的软件包是最新的。

:::

## 定义工具

如果我们希望我们的工具区分消息内容和其他工件，我们需要在定义工具时指定 `response_format="content_and_artifact"`，并确保返回一个元组 (content, artifact)：

```python
%pip install -qU "langchain-core>=0.2.19"
```

```python
import random
from typing import List, Tuple

from langchain_core.tools import tool


@tool(response_format="content_and_artifact")
def generate_random_ints(min: int, max: int, size: int) -> Tuple[str, List[int]]:
    """Generate size random ints in the range [min, max]."""
    array = [random.randint(min, max) for _ in range(size)]
    content = f"Successfully generated array of {size} random ints in [{min}, {max}]."
    return content, array
```

## 使用 ToolCall 调用工具

如果我们仅使用工具参数直接调用我们的工具，您会注意到我们只会收到工具输出的内容部分：


```python
generate_random_ints.invoke({"min": 0, "max": 9, "size": 10})
```



```output
'Successfully generated array of 10 random ints in [0, 9].'
```

```output
Failed to batch ingest runs: LangSmithRateLimitError('Rate limit exceeded for https://api.smith.langchain.com/runs/batch. HTTPError(\'429 Client Error: Too Many Requests for url: https://api.smith.langchain.com/runs/batch\', \'{"detail":"Monthly unique traces usage limit exceeded"}\')')
```
为了同时获取内容和工件，我们需要使用 ToolCall 调用我们的模型（这只是一个包含 "name"、"args"、"id" 和 "type" 键的字典），它包含生成 ToolMessage 所需的额外信息，例如工具调用 ID：


```python
generate_random_ints.invoke(
    {
        "name": "generate_random_ints",
        "args": {"min": 0, "max": 9, "size": 10},
        "id": "123",  # required
        "type": "tool_call",  # required
    }
)
```



```output
ToolMessage(content='Successfully generated array of 10 random ints in [0, 9].', name='generate_random_ints', tool_call_id='123', artifact=[2, 8, 0, 6, 0, 0, 1, 5, 0, 0])
```

## 使用模型

使用 [tool-calling model](/docs/how_to/tool_calling/)，我们可以轻松地使用模型调用我们的工具并生成 ToolMessages：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs
  customVarName="llm"
/>


```python
llm_with_tools = llm.bind_tools([generate_random_ints])

ai_msg = llm_with_tools.invoke("generate 6 positive ints less than 25")
ai_msg.tool_calls
```



```output
[{'name': 'generate_random_ints',
  'args': {'min': 1, 'max': 24, 'size': 6},
  'id': 'toolu_01EtALY3Wz1DVYhv1TLvZGvE',
  'type': 'tool_call'}]
```



```python
generate_random_ints.invoke(ai_msg.tool_calls[0])
```



```output
ToolMessage(content='Successfully generated array of 6 random ints in [1, 24].', name='generate_random_ints', tool_call_id='toolu_01EtALY3Wz1DVYhv1TLvZGvE', artifact=[2, 20, 23, 8, 1, 15])
```


如果我们只传入工具调用参数，我们将只获得内容：


```python
generate_random_ints.invoke(ai_msg.tool_calls[0]["args"])
```



```output
'Successfully generated array of 6 random ints in [1, 24].'
```


如果我们想要声明性地创建一个链，我们可以这样做：


```python
from operator import attrgetter

chain = llm_with_tools | attrgetter("tool_calls") | generate_random_ints.map()

chain.invoke("give me a random number between 1 and 5")
```



```output
[ToolMessage(content='Successfully generated array of 1 random ints in [1, 5].', name='generate_random_ints', tool_call_id='toolu_01FwYhnkwDPJPbKdGq4ng6uD', artifact=[5])]
```

## 从 BaseTool 类创建对象

如果您想直接创建一个 BaseTool 对象，而不是用 `@tool` 装饰一个函数，可以这样做：

```python
from langchain_core.tools import BaseTool


class GenerateRandomFloats(BaseTool):
    name: str = "generate_random_floats"
    description: str = "在范围 [min, max] 内生成 size 个随机浮点数。"
    response_format: str = "content_and_artifact"

    ndigits: int = 2

    def _run(self, min: float, max: float, size: int) -> Tuple[str, List[float]]:
        range_ = max - min
        array = [
            round(min + (range_ * random.random()), ndigits=self.ndigits)
            for _ in range(size)
        ]
        content = f"生成了 {size} 个浮点数在 [{min}, {max}]，四舍五入到 {self.ndigits} 位小数。"
        return content, array

    # 可选地定义一个等效的异步方法

    # async def _arun(self, min: float, max: float, size: int) -> Tuple[str, List[float]]:
    #     ...
```


```python
rand_gen = GenerateRandomFloats(ndigits=4)
rand_gen.invoke({"min": 0.1, "max": 3.3333, "size": 3})
```



```output
'生成了 3 个浮点数在 [0.1, 3.3333]，四舍五入到 4 位小数。'
```



```python
rand_gen.invoke(
    {
        "name": "generate_random_floats",
        "args": {"min": 0.1, "max": 3.3333, "size": 3},
        "id": "123",
        "type": "tool_call",
    }
)
```



```output
ToolMessage(content='生成了 3 个浮点数在 [0.1, 3.3333]，四舍五入到 4 位小数。', name='generate_random_floats', tool_call_id='123', artifact=[1.5789, 2.464, 2.2719])
```