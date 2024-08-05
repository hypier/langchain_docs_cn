---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/tools_human.ipynb
---

# 如何为工具添加人工干预

有些工具我们不信任模型单独执行。在这种情况下，我们可以要求在调用工具之前获得人工批准。

:::info

本指南展示了一种简单的方法，用于在 jupyter notebook 或终端中为代码运行添加人工干预。

要构建生产应用程序，您需要做更多工作以适当地跟踪应用程序状态。

我们建议使用 `langgraph` 来支持这种能力。有关更多详细信息，请参阅此 [指南](https://langchain-ai.github.io/langgraph/how-tos/human-in-the-loop/)。
:::

## 设置

我们需要安装以下软件包：


```python
%pip install --upgrade --quiet langchain
```

并设置这些环境变量：


```python
import getpass
import os

# 如果您想使用 LangSmith，请取消注释以下内容：
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 链

让我们创建几个简单的（虚拟）工具和一个工具调用链：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm"/>


```python
from typing import Dict, List

from langchain_core.messages import AIMessage
from langchain_core.runnables import Runnable, RunnablePassthrough
from langchain_core.tools import tool


@tool
def count_emails(last_n_days: int) -> int:
    """Multiply two integers together."""
    return last_n_days * 2


@tool
def send_email(message: str, recipient: str) -> str:
    "Add two integers."
    return f"Successfully sent email to {recipient}."


tools = [count_emails, send_email]
llm_with_tools = llm.bind_tools(tools)


def call_tools(msg: AIMessage) -> List[Dict]:
    """Simple sequential tool calling helper."""
    tool_map = {tool.name: tool for tool in tools}
    tool_calls = msg.tool_calls.copy()
    for tool_call in tool_calls:
        tool_call["output"] = tool_map[tool_call["name"]].invoke(tool_call["args"])
    return tool_calls


chain = llm_with_tools | call_tools
chain.invoke("how many emails did i get in the last 5 days?")
```



```output
[{'name': 'count_emails',
  'args': {'last_n_days': 5},
  'id': 'toolu_01QYZdJ4yPiqsdeENWHqioFW',
  'output': 10}]
```

## 添加人工审批

让我们在链中添加一个步骤，询问一个人是否批准或拒绝高呼叫请求。

在拒绝时，该步骤将引发异常，这将停止链中其余部分的执行。


```python
import json


class NotApproved(Exception):
    """自定义异常."""


def human_approval(msg: AIMessage) -> AIMessage:
    """负责传递其输入或引发异常。

    Args:
        msg: 聊天模型的输出

    Returns:
        msg: msg 的原始输出
    """
    tool_strs = "\n\n".join(
        json.dumps(tool_call, indent=2) for tool_call in msg.tool_calls
    )
    input_msg = (
        f"您是否批准以下工具调用\n\n{tool_strs}\n\n"
        "除 'Y'/'Yes'（不区分大小写）之外的任何内容都将被视为否。\n >>>"
    )
    resp = input(input_msg)
    if resp.lower() not in ("yes", "y"):
        raise NotApproved(f"工具调用未获得批准:\n\n{tool_strs}")
    return msg
```


```python
chain = llm_with_tools | human_approval | call_tools
chain.invoke("我在过去 5 天内收到了多少封电子邮件？")
```
```output
您是否批准以下工具调用

{
  "name": "count_emails",
  "args": {
    "last_n_days": 5
  },
  "id": "toolu_01WbD8XeMoQaRFtsZezfsHor"
}

除 'Y'/'Yes'（不区分大小写）之外的任何内容都将被视为否。
 >>> yes
```


```output
[{'name': 'count_emails',
  'args': {'last_n_days': 5},
  'id': 'toolu_01WbD8XeMoQaRFtsZezfsHor',
  'output': 10}]
```



```python
try:
    chain.invoke("给 sally@gmail.com 发一封邮件，内容是 'What's up homie'")
except NotApproved as e:
    print()
    print(e)
```
```output
您是否批准以下工具调用

{
  "name": "send_email",
  "args": {
    "recipient": "sally@gmail.com",
    "message": "What's up homie"
  },
  "id": "toolu_014XccHFzBiVcc9GV1harV9U"
}

除 'Y'/'Yes'（不区分大小写）之外的任何内容都将被视为否。
 >>> no
``````output

工具调用未获得批准:

{
  "name": "send_email",
  "args": {
    "recipient": "sally@gmail.com",
    "message": "What's up homie"
  },
  "id": "toolu_014XccHFzBiVcc9GV1harV9U"
}
```