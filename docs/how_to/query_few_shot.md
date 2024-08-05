---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/query_few_shot.ipynb
sidebar_position: 2
---

# 如何为查询分析添加示例

随着我们的查询分析变得越来越复杂，LLM 可能会在某些场景中难以理解它应该如何准确响应。为了提高性能，我们可以在提示中添加示例以引导 LLM。

让我们看看如何为我们在 [Quickstart](/docs/tutorials/query_analysis) 中构建的 LangChain YouTube 视频查询分析器添加示例。

## 设置
#### 安装依赖

```python
# %pip install -qU langchain-core langchain-openai
```

#### 设置环境变量

我们将在这个例子中使用 OpenAI：

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()

# 可选，取消注释以使用 LangSmith 追踪运行。在此注册： https://smith.langchain.com.
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 查询架构

我们将定义一个查询架构，期望我们的模型输出。为了使我们的查询分析更加有趣，我们将添加一个 `sub_queries` 字段，该字段包含从顶层问题派生出的更具体的问题。

```python
from typing import List, Optional

from langchain_core.pydantic_v1 import BaseModel, Field

sub_queries_description = """\
如果原始问题包含多个不同的子问题，\
或者如果有更一般的问题可以帮助回答原始问题，\
请写出所有相关子问题的列表。\
确保这个列表是全面的，涵盖原始问题的所有部分。\
如果子问题之间有冗余是可以的。\
确保子问题尽可能集中于特定主题。"""


class Search(BaseModel):
    """在关于软件库的教程视频数据库中进行搜索。"""

    query: str = Field(
        ...,
        description="应用于视频转录的主要相似性搜索查询。",
    )
    sub_queries: List[str] = Field(
        default_factory=list, description=sub_queries_description
    )
    publish_year: Optional[int] = Field(None, description="视频发布年份")
```

## 查询生成


```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI

system = """You are an expert at converting user questions into database queries. \
You have access to a database of tutorial videos about a software library for building LLM-powered applications. \
Given a question, return a list of database queries optimized to retrieve the most relevant results.

If there are acronyms or words you are not familiar with, do not try to rephrase them."""

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        MessagesPlaceholder("examples", optional=True),
        ("human", "{question}"),
    ]
)
llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
structured_llm = llm.with_structured_output(Search)
query_analyzer = {"question": RunnablePassthrough()} | prompt | structured_llm
```

让我们在没有示例的提示中尝试我们的查询分析器：


```python
query_analyzer.invoke(
    "web voyager和reflection agents之间有什么区别？两者都使用langgraph吗？"
)
```



```output
Search(query='web voyager vs reflection agents', sub_queries=['difference between web voyager and reflection agents', 'do web voyager and reflection agents use langgraph'], publish_year=None)
```

## 添加示例并调整提示

这工作得相当不错，但我们可能希望更进一步分解问题，以便将有关 Web Voyager 和 Reflection Agents 的查询分开。

为了调整我们的查询生成结果，我们可以在提示中添加一些输入问题和标准输出查询的示例。

```python
examples = []
```

```python
question = "什么是 chat langchain，它是 langchain 模板吗？"
query = Search(
    query="什么是 chat langchain，它是 langchain 模板吗？",
    sub_queries=["什么是 chat langchain", "什么是 langchain 模板"],
)
examples.append({"input": question, "tool_calls": [query]})
```

```python
question = "如何构建多智能体系统并从中流式传输中间步骤"
query = Search(
    query="如何构建多智能体系统并从中流式传输中间步骤",
    sub_queries=[
        "如何构建多智能体系统",
        "如何从多智能体系统中流式传输中间步骤",
        "如何流式传输中间步骤",
    ],
)

examples.append({"input": question, "tool_calls": [query]})
```

```python
question = "LangChain agents 和 LangGraph 有什么区别？"
query = Search(
    query="LangChain agents 和 LangGraph 之间有什么区别？如何部署它们？",
    sub_queries=[
        "什么是 LangChain agents",
        "什么是 LangGraph",
        "如何部署 LangChain agents",
        "如何部署 LangGraph",
    ],
)
examples.append({"input": question, "tool_calls": [query]})
```

现在我们需要更新我们的提示模板和链，以便在每个提示中包含示例。由于我们正在使用 OpenAI 的函数调用，我们需要进行一些额外的结构化，以便将示例输入和输出发送到模型。我们将创建一个 `tool_example_to_messages` 辅助函数来处理这个问题：

```python
import uuid
from typing import Dict

from langchain_core.messages import (
    AIMessage,
    BaseMessage,
    HumanMessage,
    SystemMessage,
    ToolMessage,
)


def tool_example_to_messages(example: Dict) -> List[BaseMessage]:
    messages: List[BaseMessage] = [HumanMessage(content=example["input"])]
    openai_tool_calls = []
    for tool_call in example["tool_calls"]:
        openai_tool_calls.append(
            {
                "id": str(uuid.uuid4()),
                "type": "function",
                "function": {
                    "name": tool_call.__class__.__name__,
                    "arguments": tool_call.json(),
                },
            }
        )
    messages.append(
        AIMessage(content="", additional_kwargs={"tool_calls": openai_tool_calls})
    )
    tool_outputs = example.get("tool_outputs") or [
        "您已正确调用此工具。"
    ] * len(openai_tool_calls)
    for output, tool_call in zip(tool_outputs, openai_tool_calls):
        messages.append(ToolMessage(content=output, tool_call_id=tool_call["id"]))
    return messages


example_msgs = [msg for ex in examples for msg in tool_example_to_messages(ex)]
```

```python
from langchain_core.prompts import MessagesPlaceholder

query_analyzer_with_examples = (
    {"question": RunnablePassthrough()}
    | prompt.partial(examples=example_msgs)
    | structured_llm
)
```

```python
query_analyzer_with_examples.invoke(
    "web voyager 和 reflection agents 之间有什么区别？两者都使用 langgraph 吗？"
)
```

```output
Search(query='web voyager 和 reflection agents 之间的区别，两者都使用 LangGraph 吗？', sub_queries=['什么是 Web Voyager', '什么是 Reflection agents', 'Web Voyager 和 Reflection agents 是否使用 LangGraph'], publish_year=None)
```

由于我们的示例，我们得到了稍微更分解的搜索查询。通过更多的提示工程和示例调整，我们可以进一步改善查询生成。

您可以看到，示例作为消息传递给模型，具体请参见 [LangSmith trace](https://smith.langchain.com/public/aeaaafce-d2b1-4943-9a61-bc954e8fc6f2/r)。