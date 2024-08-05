---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/tool_calling_parallel.ipynb
---

# 如何禁用并行工具调用

:::info OpenAI-specific

此 API 目前仅支持 OpenAI。

:::

OpenAI 工具调用默认情况下以并行方式执行工具调用。这意味着，如果我们问一个问题，例如“东京、纽约和芝加哥的天气如何？”并且我们有一个获取天气的工具，它将并行调用该工具 3 次。我们可以通过使用 ``parallel_tool_call`` 参数强制它仅调用单个工具一次。

首先，让我们设置我们的工具和模型：

```python
from langchain_core.tools import tool


@tool
def add(a: int, b: int) -> int:
    """Adds a and b."""
    return a + b


@tool
def multiply(a: int, b: int) -> int:
    """Multiplies a and b."""
    return a * b


tools = [add, multiply]
```

```python
import os
from getpass import getpass

from langchain_openai import ChatOpenAI

os.environ["OPENAI_API_KEY"] = getpass()

llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
```

现在让我们快速展示禁用并行工具调用的工作原理：

```python
llm_with_tools = llm.bind_tools(tools, parallel_tool_calls=False)
llm_with_tools.invoke("Please call the first tool two times").tool_calls
```

```output
[{'name': 'add',
  'args': {'a': 2, 'b': 2},
  'id': 'call_Hh4JOTCDM85Sm9Pr84VKrWu5'}]
```

正如我们所看到的，尽管我们明确告诉模型调用一个工具两次，但通过禁用并行工具调用，模型被限制为仅调用一次。