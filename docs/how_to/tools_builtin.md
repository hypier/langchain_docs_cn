---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/tools_builtin.ipynb
sidebar_position: 4
sidebar_class_name: hidden
---

# 如何使用内置工具和工具包

:::info 先决条件

本指南假设您熟悉以下概念：

- [LangChain Tools](/docs/concepts/#tools)
- [LangChain Toolkits](/docs/concepts/#tools)

:::

## 工具

LangChain 有大量的第三方工具。请访问 [工具集成](/docs/integrations/tools/) 查看可用工具的列表。

:::重要

使用第三方工具时，请确保您了解该工具的工作原理及其权限。请仔细阅读其文档，并检查从安全角度来看是否需要您提供任何信息。有关更多信息，请参见我们的 [安全](https://python.langchain.com/v0.2/docs/security/) 指南。

:::

让我们试一下 [维基百科集成](/docs/integrations/tools/wikipedia/)。


```python
!pip install -qU wikipedia
```


```python
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper

api_wrapper = WikipediaAPIWrapper(top_k_results=1, doc_content_chars_max=100)
tool = WikipediaQueryRun(api_wrapper=api_wrapper)

print(tool.invoke({"query": "langchain"}))
```
```output
Page: LangChain
Summary: LangChain is a framework designed to simplify the creation of applications
```
该工具具有以下默认设置：


```python
print(f"Name: {tool.name}")
print(f"Description: {tool.description}")
print(f"args schema: {tool.args}")
print(f"returns directly?: {tool.return_direct}")
```
```output
Name: wiki-tool
Description: look up things in wikipedia
args schema: {'query': {'title': 'Query', 'description': 'query to look up in Wikipedia, should be 3 or less words', 'type': 'string'}}
returns directly?: True
```

## 自定义默认工具
我们还可以修改内置参数的名称、描述和 JSON 架构。

在定义参数的 JSON 架构时，重要的是输入保持与函数相同，因此不应更改。但您可以轻松为每个输入定义自定义描述。


```python
from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
from langchain_core.pydantic_v1 import BaseModel, Field


class WikiInputs(BaseModel):
    """Inputs to the wikipedia tool."""

    query: str = Field(
        description="query to look up in Wikipedia, should be 3 or less words"
    )


tool = WikipediaQueryRun(
    name="wiki-tool",
    description="look up things in wikipedia",
    args_schema=WikiInputs,
    api_wrapper=api_wrapper,
    return_direct=True,
)

print(tool.run("langchain"))
```
```output
Page: LangChain
Summary: LangChain is a framework designed to simplify the creation of applications
```

```python
print(f"Name: {tool.name}")
print(f"Description: {tool.description}")
print(f"args schema: {tool.args}")
print(f"returns directly?: {tool.return_direct}")
```
```output
Name: wiki-tool
Description: look up things in wikipedia
args schema: {'query': {'title': 'Query', 'description': 'query to look up in Wikipedia, should be 3 or less words', 'type': 'string'}}
returns directly?: True
```

## 如何使用内置工具包

工具包是为特定任务设计的一组工具集合。它们具有便捷的加载方式。

要查看可用的现成工具包的完整列表，请访问 [Integrations](/docs/integrations/toolkits/)。

所有工具包都公开一个 `get_tools` 方法，该方法返回工具列表。

通常你应该这样使用它们：

```python
# Initialize a toolkit
toolkit = ExampleTookit(...)

# Get list of tools
tools = toolkit.get_tools()
```