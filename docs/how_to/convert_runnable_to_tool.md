---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/convert_runnable_to_tool.ipynb
---

# 如何将 Runnables 转换为工具

:::info 前提条件

本指南假定您熟悉以下概念：

- [Runnables](/docs/concepts#runnable-interface)
- [Tools](/docs/concepts#tools)
- [Agents](/docs/tutorials/agents)

:::

在这里，我们将演示如何将 LangChain `Runnable` 转换为可以被代理、链或聊天模型使用的工具。

## 依赖项

**注意**：本指南要求 `langchain-core` >= 0.2.13。我们还将使用 [OpenAI](/docs/integrations/platforms/openai/) 进行嵌入，但任何 LangChain 嵌入都应该足够。我们将使用一个简单的 [LangGraph](https://langchain-ai.github.io/langgraph/) 代理进行演示。

```python
%%capture --no-stderr
%pip install -U langchain-core langchain-openai langgraph
```

LangChain [工具](/docs/concepts#tools) 是代理、链或聊天模型用来与外界交互的接口。有关工具调用、内置工具、自定义工具及更多信息的操作指南，请参见 [这里](/docs/how_to/#tools)。

LangChain 工具——[BaseTool](https://api.python.langchain.com/en/latest/tools/langchain_core.tools.BaseTool.html) 的实例——是具有额外约束的 [可运行对象](/docs/concepts/#runnable-interface)，使其能够被语言模型有效调用：

- 它们的输入被限制为可序列化，具体为字符串和 Python `dict` 对象；
- 它们包含指示如何以及何时使用的名称和描述；
- 它们可能包含详细的 [args_schema](https://python.langchain.com/v0.2/docs/how_to/custom_tools/) 用于其参数。也就是说，虽然一个工具（作为 `Runnable`）可能接受单个 `dict` 输入，但填充 `dict` 所需的特定键和类型信息应在 `args_schema` 中指定。

接受字符串或 `dict` 输入的可运行对象可以使用 [as_tool](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.as_tool) 方法转换为工具，该方法允许为参数指定名称、描述和额外的模式信息。

## 基本用法

使用类型为 `dict` 的输入：

```python
from typing import List

from langchain_core.runnables import RunnableLambda
from typing_extensions import TypedDict


class Args(TypedDict):
    a: int
    b: List[int]


def f(x: Args) -> str:
    return str(x["a"] * max(x["b"]))


runnable = RunnableLambda(f)
as_tool = runnable.as_tool(
    name="My tool",
    description="使用工具的说明。",
)
```


```python
print(as_tool.description)

as_tool.args_schema.schema()
```
```output
使用工具的说明。
```


```output
{'title': 'My tool',
 'type': 'object',
 'properties': {'a': {'title': 'A', 'type': 'integer'},
  'b': {'title': 'B', 'type': 'array', 'items': {'type': 'integer'}}},
 'required': ['a', 'b']}
```



```python
as_tool.invoke({"a": 3, "b": [1, 2]})
```



```output
'6'
```


没有类型信息时，可以通过 `arg_types` 指定参数类型：

```python
from typing import Any, Dict


def g(x: Dict[str, Any]) -> str:
    return str(x["a"] * max(x["b"]))


runnable = RunnableLambda(g)
as_tool = runnable.as_tool(
    name="My tool",
    description="使用工具的说明。",
    arg_types={"a": int, "b": List[int]},
)
```

或者，可以通过直接传递所需的 [args_schema](https://api.python.langchain.com/en/latest/tools/langchain_core.tools.BaseTool.html#langchain_core.tools.BaseTool.args_schema) 完全指定模式：

```python
from langchain_core.pydantic_v1 import BaseModel, Field


class GSchema(BaseModel):
    """对整数和整数列表应用函数。"""

    a: int = Field(..., description="整数")
    b: List[int] = Field(..., description="整数列表")


runnable = RunnableLambda(g)
as_tool = runnable.as_tool(GSchema)
```

也支持字符串输入：

```python
def f(x: str) -> str:
    return x + "a"


def g(x: str) -> str:
    return x + "z"


runnable = RunnableLambda(f) | g
as_tool = runnable.as_tool()
```


```python
as_tool.invoke("b")
```



```output
'baz'
```

## 在代理中

接下来，我们将在一个 [代理](/docs/concepts/#agents) 应用中将 LangChain Runnables 作为工具进行整合。我们将通过以下内容进行演示：

- 一个文档 [检索器](/docs/concepts/#retrievers)；
- 一个简单的 [RAG](/docs/tutorials/rag/) 链，允许代理将相关查询委托给它。

我们首先实例化一个支持 [工具调用](/docs/how_to/tool_calling/) 的聊天模型：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />

接下来，按照 [RAG 教程](/docs/tutorials/rag/)，我们首先构建一个检索器：

```python
from langchain_core.documents import Document
from langchain_core.vectorstores import InMemoryVectorStore
from langchain_openai import OpenAIEmbeddings

documents = [
    Document(
        page_content="Dogs are great companions, known for their loyalty and friendliness.",
    ),
    Document(
        page_content="Cats are independent pets that often enjoy their own space.",
    ),
]

vectorstore = InMemoryVectorStore.from_documents(
    documents, embedding=OpenAIEmbeddings()
)

retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 1},
)
```

接下来，我们使用一个简单的预构建 [LangGraph 代理](https://python.langchain.com/v0.2/docs/tutorials/agents/) 并为其提供工具：

```python
from langgraph.prebuilt import create_react_agent

tools = [
    retriever.as_tool(
        name="pet_info_retriever",
        description="Get information about pets.",
    )
]
agent = create_react_agent(llm, tools)
```

```python
for chunk in agent.stream({"messages": [("human", "What are dogs known for?")]}):
    print(chunk)
    print("----")
```
```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_W8cnfOjwqEn4cFcg19LN9mYD', 'function': {'arguments': '{"__arg1":"dogs"}', 'name': 'pet_info_retriever'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 19, 'prompt_tokens': 60, 'total_tokens': 79}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-d7f81de9-1fb7-4caf-81ed-16dcdb0b2ab4-0', tool_calls=[{'name': 'pet_info_retriever', 'args': {'__arg1': 'dogs'}, 'id': 'call_W8cnfOjwqEn4cFcg19LN9mYD'}], usage_metadata={'input_tokens': 60, 'output_tokens': 19, 'total_tokens': 79})]}}
----
{'tools': {'messages': [ToolMessage(content="[Document(id='86f835fe-4bbe-4ec6-aeb4-489a8b541707', page_content='Dogs are great companions, known for their loyalty and friendliness.')]", name='pet_info_retriever', tool_call_id='call_W8cnfOjwqEn4cFcg19LN9mYD')]}}
----
{'agent': {'messages': [AIMessage(content='Dogs are known for being great companions, known for their loyalty and friendliness.', response_metadata={'token_usage': {'completion_tokens': 18, 'prompt_tokens': 134, 'total_tokens': 152}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-9ca5847a-a5eb-44c0-a774-84cc2c5bbc5b-0', usage_metadata={'input_tokens': 134, 'output_tokens': 18, 'total_tokens': 152})]}}
----
```
请查看上述运行的 [LangSmith 跟踪](https://smith.langchain.com/public/44e438e3-2faf-45bd-b397-5510fc145eb9/r)。

进一步，我们可以创建一个简单的 [RAG](/docs/tutorials/rag/) 链，该链接受一个额外的参数——在这里是答案的“风格”。

```python
from operator import itemgetter

from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough

system_prompt = """
You are an assistant for question-answering tasks.
Use the below context to answer the question. If
you don't know the answer, say you don't know.
Use three sentences maximum and keep the answer
concise.

Answer in the style of {answer_style}.

Question: {question}

Context: {context}
"""

prompt = ChatPromptTemplate.from_messages([("system", system_prompt)])

rag_chain = (
    {
        "context": itemgetter("question") | retriever,
        "question": itemgetter("question"),
        "answer_style": itemgetter("answer_style"),
    }
    | prompt
    | llm
    | StrOutputParser()
)
```

请注意，我们的链的输入模式包含所需的参数，因此它可以无须进一步说明地转换为工具：

```python
rag_chain.input_schema.schema()
```

```output
{'title': 'RunnableParallel<context,question,answer_style>Input',
 'type': 'object',
 'properties': {'question': {'title': 'Question'},
  'answer_style': {'title': 'Answer Style'}}}
```

```python
rag_tool = rag_chain.as_tool(
    name="pet_expert",
    description="Get information about pets.",
)
```

下面我们再次调用代理。请注意，代理在其 `tool_calls` 中填充所需的参数：

```python
agent = create_react_agent(llm, [rag_tool])

for chunk in agent.stream(
    {"messages": [("human", "What would a pirate say dogs are known for?")]}
):
    print(chunk)
    print("----")
```
```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_17iLPWvOD23zqwd1QVQ00Y63', 'function': {'arguments': '{"question":"What are dogs known for according to pirates?","answer_style":"quote"}', 'name': 'pet_expert'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 28, 'prompt_tokens': 59, 'total_tokens': 87}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-7fef44f3-7bba-4e63-8c51-2ad9c5e65e2e-0', tool_calls=[{'name': 'pet_expert', 'args': {'question': 'What are dogs known for according to pirates?', 'answer_style': 'quote'}, 'id': 'call_17iLPWvOD23zqwd1QVQ00Y63'}], usage_metadata={'input_tokens': 59, 'output_tokens': 28, 'total_tokens': 87})]}}
----
{'tools': {'messages': [ToolMessage(content='"Dogs are known for their loyalty and friendliness, making them great companions for pirates on long sea voyages."', name='pet_expert', tool_call_id='call_17iLPWvOD23zqwd1QVQ00Y63')]}}
----
{'agent': {'messages': [AIMessage(content='According to pirates, dogs are known for their loyalty and friendliness, making them great companions for pirates on long sea voyages.', response_metadata={'token_usage': {'completion_tokens': 27, 'prompt_tokens': 119, 'total_tokens': 146}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-5a30edc3-7be0-4743-b980-ca2f8cad9b8d-0', usage_metadata={'input_tokens': 119, 'output_tokens': 27, 'total_tokens': 146})]}}
----
```
请查看上述运行的 [LangSmith 跟踪](https://smith.langchain.com/public/147ae4e6-4dfb-4dd9-8ca0-5c5b954f08ac/r)。