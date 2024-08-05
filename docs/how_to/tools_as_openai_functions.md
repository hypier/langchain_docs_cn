---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/tools_as_openai_functions.ipynb
---

# 如何将工具转换为OpenAI函数

本笔记本介绍如何将LangChain工具用作OpenAI函数。

```python
%pip install -qU langchain-community langchain-openai
```

```python
from langchain_community.tools import MoveFileTool
from langchain_core.messages import HumanMessage
from langchain_core.utils.function_calling import convert_to_openai_function
from langchain_openai import ChatOpenAI
```

```python
model = ChatOpenAI(model="gpt-3.5-turbo")
```

```python
tools = [MoveFileTool()]
functions = [convert_to_openai_function(t) for t in tools]
```

```python
functions[0]
```

```output
{'name': 'move_file',
 'description': '将文件从一个位置移动或重命名到另一个位置',
 'parameters': {'type': 'object',
  'properties': {'source_path': {'description': '要移动的文件路径',
    'type': 'string'},
   'destination_path': {'description': '移动后文件的新路径',
    'type': 'string'}},
  'required': ['source_path', 'destination_path']}}
```

```python
message = model.invoke(
    [HumanMessage(content="move file foo to bar")], functions=functions
)
```

```python
message
```

```output
AIMessage(content='', additional_kwargs={'function_call': {'arguments': '{\n  "source_path": "foo",\n  "destination_path": "bar"\n}', 'name': 'move_file'}})
```

```python
message.additional_kwargs["function_call"]
```

```output
{'name': 'move_file',
 'arguments': '{\n  "source_path": "foo",\n  "destination_path": "bar"\n}'}
```

使用OpenAI聊天模型，我们还可以通过`bind_functions`自动绑定和转换类似函数的对象。

```python
model_with_functions = model.bind_functions(tools)
model_with_functions.invoke([HumanMessage(content="move file foo to bar")])
```

```output
AIMessage(content='', additional_kwargs={'function_call': {'arguments': '{\n  "source_path": "foo",\n  "destination_path": "bar"\n}', 'name': 'move_file'}})
```

或者，我们可以使用更新的OpenAI API，通过使用`ChatOpenAI.bind_tools`来使用`tools`和`tool_choice`，而不是`functions`和`function_call`：

```python
model_with_tools = model.bind_tools(tools)
model_with_tools.invoke([HumanMessage(content="move file foo to bar")])
```

```output
AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_btkY3xV71cEVAOHnNa5qwo44', 'function': {'arguments': '{\n  "source_path": "foo",\n  "destination_path": "bar"\n}', 'name': 'move_file'}, 'type': 'function'}]})
```