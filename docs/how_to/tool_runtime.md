---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/tool_runtime.ipynb
---

# 如何将运行时值传递给工具

import Prerequisites from "@theme/Prerequisites";
import Compatibility from "@theme/Compatibility";

<Prerequisites titlesAndLinks={[
  ["聊天模型", "/docs/concepts/#chat-models"],
  ["LangChain 工具", "/docs/concepts/#tools"],
  ["如何创建工具", "/docs/how_to/custom_tools"],
  ["如何使用模型调用工具", "/docs/how_to/tool_calling"],
]} />

<Compatibility packagesAndVersions={[
  ["langchain-core", "0.2.21"],
]} />

您可能需要将仅在运行时已知的值绑定到工具。例如，工具逻辑可能需要使用发出请求的用户的 ID。

大多数情况下，这些值不应由 LLM 控制。实际上，允许 LLM 控制用户 ID 可能会导致安全风险。

相反，LLM 应仅控制那些由 LLM 需要控制的工具参数，而其他参数（例如用户 ID）应由应用逻辑固定。

本指南将向您展示如何防止模型生成某些工具参数并在运行时直接注入它们。

:::info 与 LangGraph 一起使用

如果您正在使用 LangGraph，请参考 [此指南](https://langchain-ai.github.io/langgraph/how-tos/pass-run-time-values-to-tools/)
该指南展示了如何创建一个代理来跟踪特定用户的宠物喜好。
:::

我们可以将它们绑定到聊天模型，如下所示：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs
  customVarName="llm"
  fireworksParams={`model="accounts/fireworks/models/firefunction-v1", temperature=0`}
/>

## 隐藏模型的参数

我们可以使用 InjectedToolArg 注解来标记我们工具的某些参数，例如 `user_id`，表示它们在运行时被注入，意味着它们不应该由模型生成。

```python
from typing import List

from langchain_core.tools import InjectedToolArg, tool
from typing_extensions import Annotated

user_to_pets = {}


@tool(parse_docstring=True)
def update_favorite_pets(
    pets: List[str], user_id: Annotated[str, InjectedToolArg]
) -> None:
    """添加最喜欢的宠物列表。

    Args:
        pets: 要设置的最喜欢的宠物列表。
        user_id: 用户的 ID。
    """
    user_to_pets[user_id] = pets


@tool(parse_docstring=True)
def delete_favorite_pets(user_id: Annotated[str, InjectedToolArg]) -> None:
    """删除最喜欢的宠物列表。

    Args:
        user_id: 用户的 ID。
    """
    if user_id in user_to_pets:
        del user_to_pets[user_id]


@tool(parse_docstring=True)
def list_favorite_pets(user_id: Annotated[str, InjectedToolArg]) -> None:
    """列出最喜欢的宠物（如果有的话）。

    Args:
        user_id: 用户的 ID。
    """
    return user_to_pets.get(user_id, [])
```

如果我们查看这些工具的输入模式，我们会看到 user_id 仍然被列出：

```python
update_favorite_pets.get_input_schema().schema()
```

```output
{'title': 'update_favorite_petsSchema',
 'description': '添加最喜欢的宠物列表。',
 'type': 'object',
 'properties': {'pets': {'title': '宠物',
   'description': '要设置的最喜欢的宠物列表。',
   'type': 'array',
   'items': {'type': 'string'}},
  'user_id': {'title': '用户 ID',
   'description': "用户的 ID。",
   'type': 'string'}},
 'required': ['pets', 'user_id']}
```

但是如果我们查看工具调用模式，也就是传递给模型进行工具调用的模式，user_id 已被移除：

```python
update_favorite_pets.tool_call_schema.schema()
```

```output
{'title': 'update_favorite_pets',
 'description': '添加最喜欢的宠物列表。',
 'type': 'object',
 'properties': {'pets': {'title': '宠物',
   'description': '要设置的最喜欢的宠物列表。',
   'type': 'array',
   'items': {'type': 'string'}}},
 'required': ['pets']}
```

因此，当我们调用工具时，需要传入 user_id：

```python
user_id = "123"
update_favorite_pets.invoke({"pets": ["lizard", "dog"], "user_id": user_id})
print(user_to_pets)
print(list_favorite_pets.invoke({"user_id": user_id}))
```
```output
{'123': ['lizard', 'dog']}
['lizard', 'dog']
```
但是当模型调用工具时，不会生成 user_id 参数：

```python
tools = [
    update_favorite_pets,
    delete_favorite_pets,
    list_favorite_pets,
]
llm_with_tools = llm.bind_tools(tools)
ai_msg = llm_with_tools.invoke("我最喜欢的动物是猫和鹦鹉")
ai_msg.tool_calls
```

```output
[{'name': 'update_favorite_pets',
  'args': {'pets': ['cats', 'parrots']},
  'id': 'call_W3cn4lZmJlyk8PCrKN4PRwqB',
  'type': 'tool_call'}]
```

## 在运行时注入参数

如果我们想实际执行使用模型生成的工具调用的工具，我们需要自己注入 user_id：


```python
from copy import deepcopy

from langchain_core.runnables import chain


@chain
def inject_user_id(ai_msg):
    tool_calls = []
    for tool_call in ai_msg.tool_calls:
        tool_call_copy = deepcopy(tool_call)
        tool_call_copy["args"]["user_id"] = user_id
        tool_calls.append(tool_call_copy)
    return tool_calls


inject_user_id.invoke(ai_msg)
```



```output
[{'name': 'update_favorite_pets',
  'args': {'pets': ['cats', 'parrots'], 'user_id': '123'},
  'id': 'call_W3cn4lZmJlyk8PCrKN4PRwqB',
  'type': 'tool_call'}]
```


现在我们可以将模型、注入代码和实际工具串联在一起，创建一个执行工具的链：


```python
tool_map = {tool.name: tool for tool in tools}


@chain
def tool_router(tool_call):
    return tool_map[tool_call["name"]]


chain = llm_with_tools | inject_user_id | tool_router.map()
chain.invoke("my favorite animals are cats and parrots")
```



```output
[ToolMessage(content='null', name='update_favorite_pets', tool_call_id='call_HUyF6AihqANzEYxQnTUKxkXj')]
```


查看 user_to_pets 字典，我们可以看到它已被更新为包括猫和鹦鹉：


```python
user_to_pets
```



```output
{'123': ['cats', 'parrots']}
```

## 其他注释 args 的方法

这里有几种其他注释我们工具 args 的方法：


```python
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_core.tools import BaseTool


class UpdateFavoritePetsSchema(BaseModel):
    """更新最喜欢的宠物列表"""

    pets: List[str] = Field(..., description="要设置的最喜欢的宠物列表。")
    user_id: Annotated[str, InjectedToolArg] = Field(..., description="用户的 ID。")


@tool(args_schema=UpdateFavoritePetsSchema)
def update_favorite_pets(pets, user_id):
    user_to_pets[user_id] = pets


update_favorite_pets.get_input_schema().schema()
```



```output
{'title': 'UpdateFavoritePetsSchema',
 'description': '更新最喜欢的宠物列表',
 'type': 'object',
 'properties': {'pets': {'title': '宠物',
   'description': '要设置的最喜欢的宠物列表。',
   'type': 'array',
   'items': {'type': 'string'}},
  'user_id': {'title': '用户 ID',
   'description': "用户的 ID。",
   'type': 'string'}},
 'required': ['pets', 'user_id']}
```



```python
update_favorite_pets.tool_call_schema.schema()
```



```output
{'title': 'update_favorite_pets',
 'description': '更新最喜欢的宠物列表',
 'type': 'object',
 'properties': {'pets': {'title': '宠物',
   'description': '要设置的最喜欢的宠物列表。',
   'type': 'array',
   'items': {'type': 'string'}}},
 'required': ['pets']}
```



```python
from typing import Optional, Type


class UpdateFavoritePets(BaseTool):
    name: str = "update_favorite_pets"
    description: str = "更新最喜欢的宠物列表"
    args_schema: Optional[Type[BaseModel]] = UpdateFavoritePetsSchema

    def _run(self, pets, user_id):
        user_to_pets[user_id] = pets


UpdateFavoritePets().get_input_schema().schema()
```



```output
{'title': 'UpdateFavoritePetsSchema',
 'description': '更新最喜欢的宠物列表',
 'type': 'object',
 'properties': {'pets': {'title': '宠物',
   'description': '要设置的最喜欢的宠物列表。',
   'type': 'array',
   'items': {'type': 'string'}},
  'user_id': {'title': '用户 ID',
   'description': "用户的 ID。",
   'type': 'string'}},
 'required': ['pets', 'user_id']}
```



```python
UpdateFavoritePets().tool_call_schema.schema()
```



```output
{'title': 'update_favorite_pets',
 'description': '更新最喜欢的宠物列表',
 'type': 'object',
 'properties': {'pets': {'title': '宠物',
   'description': '要设置的最喜欢的宠物列表。',
   'type': 'array',
   'items': {'type': 'string'}}},
 'required': ['pets']}
```



```python
class UpdateFavoritePets2(BaseTool):
    name: str = "update_favorite_pets"
    description: str = "更新最喜欢的宠物列表"

    def _run(self, pets: List[str], user_id: Annotated[str, InjectedToolArg]) -> None:
        user_to_pets[user_id] = pets


UpdateFavoritePets2().get_input_schema().schema()
```



```output
{'title': 'update_favorite_petsSchema',
 'description': '使用该工具。\n\n将 run_manager: Optional[CallbackManagerForToolRun] = None\n添加到子实现中以启用追踪。',
 'type': 'object',
 'properties': {'pets': {'title': '宠物',
   'type': 'array',
   'items': {'type': 'string'}},
  'user_id': {'title': '用户 ID', 'type': 'string'}},
 'required': ['pets', 'user_id']}
```



```python
UpdateFavoritePets2().tool_call_schema.schema()
```



```output
{'title': 'update_favorite_pets',
 'description': '更新最喜欢的宠物列表',
 'type': 'object',
 'properties': {'pets': {'title': '宠物',
   'type': 'array',
   'items': {'type': 'string'}}},
 'required': ['pets']}
```