---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/migrate_agent.ipynb
keywords: [create_react_agent, create_react_agent()]
---

# 如何从遗留的 LangChain 代理迁移到 LangGraph

:::info 前提条件

本指南假设您对以下概念有所了解：
- [代理](/docs/concepts/#agents)
- [LangGraph](https://langchain-ai.github.io/langgraph/)
- [工具调用](/docs/how_to/tool_calling/)

:::

在这里，我们重点介绍如何从遗留的 LangChain 代理迁移到更灵活的 [LangGraph](https://langchain-ai.github.io/langgraph/) 代理。
LangChain 代理（特别是 [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor)）具有多个配置参数。
在本笔记本中，我们将展示这些参数如何映射到 LangGraph 的反应代理执行器，使用 [create_react_agent](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) 预构建辅助方法。

#### 前提条件

本指南使用 OpenAI 作为 LLM。安装依赖项以运行。

```python
%%capture --no-stderr
%pip install -U langgraph langchain langchain-openai
```

然后，设置您的 OpenAI API 密钥。

```python
import os

os.environ["OPENAI_API_KEY"] = "sk-..."
```

## 基本用法

对于工具调用 ReAct 风格代理的基本创建和使用，功能是相同的。首先，让我们定义一个模型和工具，然后使用这些工具创建一个代理。


```python
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o")


@tool
def magic_function(input: int) -> int:
    """Applies a magic function to an input."""
    return input + 2


tools = [magic_function]


query = "what is the value of magic_function(3)?"
```

对于 LangChain [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor)，我们定义一个带有代理草稿的占位符的提示。代理可以如下调用：


```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant"),
        ("human", "{input}"),
        # Placeholders fill up a **list** of messages
        ("placeholder", "{agent_scratchpad}"),
    ]
)


agent = create_tool_calling_agent(model, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)

agent_executor.invoke({"input": query})
```



```output
{'input': 'what is the value of magic_function(3)?',
 'output': 'The value of `magic_function(3)` is 5.'}
```


LangGraph 的 [react agent executor](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) 管理一个由消息列表定义的状态。它将继续处理该列表，直到代理的输出中没有工具调用。为了启动它，我们输入一系列消息。输出将包含图的整个状态——在这种情况下，是对话历史。




```python
from langgraph.prebuilt import create_react_agent

app = create_react_agent(model, tools)


messages = app.invoke({"messages": [("human", query)]})
{
    "input": query,
    "output": messages["messages"][-1].content,
}
```



```output
{'input': 'what is the value of magic_function(3)?',
 'output': 'The value of `magic_function(3)` is 5.'}
```



```python
message_history = messages["messages"]

new_query = "Pardon?"

messages = app.invoke({"messages": message_history + [("human", new_query)]})
{
    "input": new_query,
    "output": messages["messages"][-1].content,
}
```



```output
{'input': 'Pardon?',
 'output': 'The value you get when you apply `magic_function` to the input 3 is 5.'}
```

## 提示模板

使用遗留的 LangChain 代理时，您必须传入一个提示模板。您可以使用它来控制代理。

在 LangGraph [react agent executor](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) 中，默认情况下没有提示。您可以通过几种方式实现对代理的类似控制：

1. 作为输入传入系统消息
2. 用系统消息初始化代理
3. 用一个函数初始化代理，以在传递给模型之前转换消息。

让我们看看下面的所有这些。我们将传入自定义指令以使代理以西班牙语响应。

首先，使用 `AgentExecutor`：

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant. Respond only in Spanish."),
        ("human", "{input}"),
        # Placeholders fill up a **list** of messages
        ("placeholder", "{agent_scratchpad}"),
    ]
)

agent = create_tool_calling_agent(model, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)

agent_executor.invoke({"input": query})
```

```output
{'input': 'what is the value of magic_function(3)?',
 'output': 'El valor de `magic_function(3)` es 5.'}
```

现在，让我们向 [react agent executor](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) 传递一个自定义系统消息。

LangGraph 的预构建 `create_react_agent` 不直接将提示模板作为参数传入，而是接受一个 [`state_modifier`](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) 参数。此参数在调用 llm 之前修改图形状态，可以是以下四个值之一：

- 一个 `SystemMessage`，它会添加到消息列表的开头。
- 一个 `string`，它会被转换为 `SystemMessage` 并添加到消息列表的开头。
- 一个 `Callable`，它应该接收完整的图形状态。输出将传递给语言模型。
- 或者一个 [`Runnable`](/docs/concepts/#langchain-expression-language-lcel)，它应该接收完整的图形状态。输出将传递给语言模型。

以下是其实际效果：

```python
from langchain_core.messages import SystemMessage
from langgraph.prebuilt import create_react_agent

system_message = "You are a helpful assistant. Respond only in Spanish."
# This could also be a SystemMessage object
# system_message = SystemMessage(content="You are a helpful assistant. Respond only in Spanish.")

app = create_react_agent(model, tools, state_modifier=system_message)

messages = app.invoke({"messages": [("user", query)]})
```

我们还可以传入一个任意函数。此函数应接收一组消息并输出一组消息。
我们可以在这里进行各种任意的消息格式化。在这种情况下，我们只需在消息列表的开头添加一个 SystemMessage。

```python
from langgraph.prebuilt import create_react_agent
from langgraph.prebuilt.chat_agent_executor import AgentState

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant. Respond only in Spanish."),
        ("placeholder", "{messages}"),
    ]
)

def _modify_state_messages(state: AgentState):
    return prompt.invoke({"messages": state["messages"]}).to_messages() + [
        ("user", "Also say 'Pandamonium!' after the answer.")
    ]

app = create_react_agent(model, tools, state_modifier=_modify_state_messages)

messages = app.invoke({"messages": [("human", query)]})
print(
    {
        "input": query,
        "output": messages["messages"][-1].content,
    }
)
```
```output
{'input': 'what is the value of magic_function(3)?', 'output': 'El valor de magic_function(3) es 5. ¡Pandamonium!'}
```

## 内存

### 在 LangChain 中

使用 LangChain 的 [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.iter)，您可以添加聊天 [Memory](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.memory)，使其能够进行多轮对话。

```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o")
memory = InMemoryChatMessageHistory(session_id="test-session")
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        # First put the history
        ("placeholder", "{chat_history}"),
        # Then the new input
        ("human", "{input}"),
        # Finally the scratchpad
        ("placeholder", "{agent_scratchpad}"),
    ]
)


@tool
def magic_function(input: int) -> int:
    """Applies a magic function to an input."""
    return input + 2


tools = [magic_function]


agent = create_tool_calling_agent(model, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)

agent_with_chat_history = RunnableWithMessageHistory(
    agent_executor,
    # This is needed because in most real world scenarios, a session id is needed
    # It isn't really used here because we are using a simple in memory ChatMessageHistory
    lambda session_id: memory,
    input_messages_key="input",
    history_messages_key="chat_history",
)

config = {"configurable": {"session_id": "test-session"}}
print(
    agent_with_chat_history.invoke(
        {"input": "Hi, I'm polly! What's the output of magic_function of 3?"}, config
    )["output"]
)
print("---")
print(agent_with_chat_history.invoke({"input": "Remember my name?"}, config)["output"])
print("---")
print(
    agent_with_chat_history.invoke({"input": "what was that output again?"}, config)[
        "output"
    ]
)
```
```output
Hi Polly! The output of the magic function for the input 3 is 5.
---
Yes, your name is Polly!
---
The output of the magic function for the input 3 is 5.
```

### 在 LangGraph 中

内存就是 [持久性](https://langchain-ai.github.io/langgraph/how-tos/persistence/)，也称为 [检查点](https://langchain-ai.github.io/langgraph/reference/checkpoints/)。

向代理添加一个 `checkpointer`，你就可以免费获得聊天内存。

```python
from langgraph.checkpoint import MemorySaver  # an in-memory checkpointer
from langgraph.prebuilt import create_react_agent

system_message = "You are a helpful assistant."
# This could also be a SystemMessage object
# system_message = SystemMessage(content="You are a helpful assistant. Respond only in Spanish.")

memory = MemorySaver()
app = create_react_agent(
    model, tools, state_modifier=system_message, checkpointer=memory
)

config = {"configurable": {"thread_id": "test-thread"}}
print(
    app.invoke(
        {
            "messages": [
                ("user", "Hi, I'm polly! What's the output of magic_function of 3?")
            ]
        },
        config,
    )["messages"][-1].content
)
print("---")
print(
    app.invoke({"messages": [("user", "Remember my name?")]}, config)["messages"][
        -1
    ].content
)
print("---")
print(
    app.invoke({"messages": [("user", "what was that output again?")]}, config)[
        "messages"
    ][-1].content
)
```
```output
Hi Polly! The output of the magic_function for the input of 3 is 5.
---
Yes, your name is Polly!
---
The output of the magic_function for the input of 3 was 5.
```

## 遍历步骤

### 在 LangChain 中

使用 LangChain 的 [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.iter)，您可以使用 [stream](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.stream)（或异步 `astream`）方法或 [iter](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.iter) 方法逐步迭代。LangGraph 支持通过 [stream](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.stream) 进行逐步迭代。

```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o")


prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        ("human", "{input}"),
        # Placeholders fill up a **list** of messages
        ("placeholder", "{agent_scratchpad}"),
    ]
)


@tool
def magic_function(input: int) -> int:
    """Applies a magic function to an input."""
    return input + 2


tools = [magic_function]

agent = create_tool_calling_agent(model, tools, prompt=prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)

for step in agent_executor.stream({"input": query}):
    print(step)
```
```output
{'actions': [ToolAgentAction(tool='magic_function', tool_input={'input': 3}, log="\nInvoking: `magic_function` with `{'input': 3}`\n\n\n", message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_1exy0rScfPmo4fy27FbQ5qJ2', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518'}, id='run-5664e138-7085-4da7-a49e-5656a87b8d78', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_1exy0rScfPmo4fy27FbQ5qJ2', 'type': 'tool_call'}], tool_call_chunks=[{'name': 'magic_function', 'args': '{"input":3}', 'id': 'call_1exy0rScfPmo4fy27FbQ5qJ2', 'index': 0, 'type': 'tool_call_chunk'}])], tool_call_id='call_1exy0rScfPmo4fy27FbQ5qJ2')], 'messages': [AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_1exy0rScfPmo4fy27FbQ5qJ2', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518'}, id='run-5664e138-7085-4da7-a49e-5656a87b8d78', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_1exy0rScfPmo4fy27FbQ5qJ2', 'type': 'tool_call'}], tool_call_chunks=[{'name': 'magic_function', 'args': '{"input":3}', 'id': 'call_1exy0rScfPmo4fy27FbQ5qJ2', 'index': 0, 'type': 'tool_call_chunk'}])]}
{'steps': [AgentStep(action=ToolAgentAction(tool='magic_function', tool_input={'input': 3}, log="\nInvoking: `magic_function` with `{'input': 3}`\n\n\n", message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_1exy0rScfPmo4fy27FbQ5qJ2', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518'}, id='run-5664e138-7085-4da7-a49e-5656a87b8d78', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_1exy0rScfPmo4fy27FbQ5qJ2', 'type': 'tool_call'}], tool_call_chunks=[{'name': 'magic_function', 'args': '{"input":3}', 'id': 'call_1exy0rScfPmo4fy27FbQ5qJ2', 'index': 0, 'type': 'tool_call_chunk'}])], tool_call_id='call_1exy0rScfPmo4fy27FbQ5qJ2'), observation=5)], 'messages': [FunctionMessage(content='5', name='magic_function')]}
{'output': 'The value of `magic_function(3)` is 5.', 'messages': [AIMessage(content='The value of `magic_function(3)` is 5.')}]
```

### 在 LangGraph 中

在 LangGraph 中，使用 [stream](https://langchain-ai.github.io/langgraph/reference/graphs/#langgraph.graph.graph.CompiledGraph.stream) 或异步的 `astream` 方法进行原生处理。

```python
from langgraph.prebuilt import create_react_agent
from langgraph.prebuilt.chat_agent_executor import AgentState

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        ("placeholder", "{messages}"),
    ]
)


def _modify_state_messages(state: AgentState):
    return prompt.invoke({"messages": state["messages"]}).to_messages()


app = create_react_agent(model, tools, state_modifier=_modify_state_messages)

for step in app.stream({"messages": [("human", query)]}, stream_mode="updates"):
    print(step)
```
```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_my9rzFSKR4T1yYKwCsfbZB8A', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 61, 'total_tokens': 75}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_bc2a86f5f5', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-dd705555-8fae-4fb1-a033-5d99a23e3c22-0', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_my9rzFSKR4T1yYKwCsfbZB8A', 'type': 'tool_call'}], usage_metadata={'input_tokens': 61, 'output_tokens': 14, 'total_tokens': 75})]}}
{'tools': {'messages': [ToolMessage(content='5', name='magic_function', tool_call_id='call_my9rzFSKR4T1yYKwCsfbZB8A')]}}
{'agent': {'messages': [AIMessage(content='The value of `magic_function(3)` is 5.', response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 84, 'total_tokens': 98}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'stop', 'logprobs': None}, id='run-698cad05-8cb2-4d08-8c2a-881e354f6cc7-0', usage_metadata={'input_tokens': 84, 'output_tokens': 14, 'total_tokens': 98})]}}
```

## `return_intermediate_steps`

### 在 LangChain 中

在 AgentExecutor 上设置此参数允许用户访问 intermediate_steps，这将代理操作（例如，工具调用）与其结果配对。

```python
agent_executor = AgentExecutor(agent=agent, tools=tools, return_intermediate_steps=True)
result = agent_executor.invoke({"input": query})
print(result["intermediate_steps"])
```
```output
[(ToolAgentAction(tool='magic_function', tool_input={'input': 3}, log="\nInvoking: `magic_function` with `{'input': 3}`\n\n\n", message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_uPZ2D1Bo5mdED3gwgaeWURrf', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518'}, id='run-a792db4a-278d-4090-82ae-904a30eada93', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_uPZ2D1Bo5mdED3gwgaeWURrf', 'type': 'tool_call'}], tool_call_chunks=[{'name': 'magic_function', 'args': '{"input":3}', 'id': 'call_uPZ2D1Bo5mdED3gwgaeWURrf', 'index': 0, 'type': 'tool_call_chunk'}])], tool_call_id='call_uPZ2D1Bo5mdED3gwgaeWURrf'), 5)]
```

### 在 LangGraph 中

默认情况下，LangGraph 中的 [react agent executor](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) 会将所有消息附加到中央状态。因此，只需查看完整状态即可轻松查看任何中间步骤。

```python
from langgraph.prebuilt import create_react_agent

app = create_react_agent(model, tools=tools)

messages = app.invoke({"messages": [("human", query)]})

messages
```

```output
{'messages': [HumanMessage(content='what is the value of magic_function(3)?', id='cd7d0f49-a0e0-425a-b2b0-603a716058ed'),
  AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_VfZ9287DuybOSrBsQH5X12xf', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 55, 'total_tokens': 69}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-a1e965cd-bf61-44f9-aec1-8aaecb80955f-0', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_VfZ9287DuybOSrBsQH5X12xf', 'type': 'tool_call'}], usage_metadata={'input_tokens': 55, 'output_tokens': 14, 'total_tokens': 69}),
  ToolMessage(content='5', name='magic_function', id='20d5c2fe-a5d8-47fa-9e04-5282642e2039', tool_call_id='call_VfZ9287DuybOSrBsQH5X12xf'),
  AIMessage(content='The value of `magic_function(3)` is 5.', response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 78, 'total_tokens': 92}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'stop', 'logprobs': None}, id='run-abf9341c-ef41-4157-935d-a3be5dfa2f41-0', usage_metadata={'input_tokens': 78, 'output_tokens': 14, 'total_tokens': 92})]}
```

## `max_iterations`

### 在 LangChain 中

`AgentExecutor` 实现了一个 `max_iterations` 参数，允许用户中止超过指定迭代次数的运行。

```python
@tool
def magic_function(input: str) -> str:
    """Applies a magic function to an input."""
    return "Sorry, there was an error. Please try again."


tools = [magic_function]
```

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant. Respond only in Spanish."),
        ("human", "{input}"),
        # Placeholders fill up a **list** of messages
        ("placeholder", "{agent_scratchpad}"),
    ]
)

agent = create_tool_calling_agent(model, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=3,
)

agent_executor.invoke({"input": query})
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Invoking: `magic_function` with `{'input': '3'}`


[0m[36;1m[1;3mSorry, there was an error. Please try again.[0m[32;1m[1;3mParece que hubo un error al intentar calcular el valor de la función mágica. ¿Te gustaría que lo intente de nuevo?[0m

[1m> Finished chain.[0m
```

```output
{'input': 'what is the value of magic_function(3)?',
 'output': 'Parece que hubo un error al intentar calcular el valor de la función mágica. ¿Te gustaría que lo intente de nuevo?'}
```

### 在 LangGraph 中

在 LangGraph 中，这通过 `recursion_limit` 配置参数进行控制。

请注意，在 `AgentExecutor` 中，“迭代”包括工具调用和执行的完整轮次。在 LangGraph 中，每一步都对递归限制产生贡献，因此我们需要将其乘以二（并加一）以获得等效结果。

如果达到递归限制，LangGraph 会引发特定的异常类型，我们可以像处理 `AgentExecutor` 一样捕获和管理该异常。

```python
from langgraph.errors import GraphRecursionError
from langgraph.prebuilt import create_react_agent

RECURSION_LIMIT = 2 * 3 + 1

app = create_react_agent(model, tools=tools)

try:
    for chunk in app.stream(
        {"messages": [("human", query)]},
        {"recursion_limit": RECURSION_LIMIT},
        stream_mode="values",
    ):
        print(chunk["messages"][-1])
except GraphRecursionError:
    print({"input": query, "output": "Agent stopped due to max iterations."})
```
```output
content='what is the value of magic_function(3)?' id='74e2d5e8-2b59-4820-979c-8d11ecfc14c2'
content='' additional_kwargs={'tool_calls': [{'id': 'call_ihtrH6IG95pDXpKluIwAgi3J', 'function': {'arguments': '{"input":"3"}', 'name': 'magic_function'}, 'type': 'function'}]} response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 55, 'total_tokens': 69}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'tool_calls', 'logprobs': None} id='run-5a35e465-8a08-43dd-ac8b-4a76dcace305-0' tool_calls=[{'name': 'magic_function', 'args': {'input': '3'}, 'id': 'call_ihtrH6IG95pDXpKluIwAgi3J', 'type': 'tool_call'}] usage_metadata={'input_tokens': 55, 'output_tokens': 14, 'total_tokens': 69}
content='Sorry, there was an error. Please try again.' name='magic_function' id='8c37c19b-3586-46b1-aab9-a045786801a2' tool_call_id='call_ihtrH6IG95pDXpKluIwAgi3J'
content='It seems there was an error in processing the request. Let me try again.' additional_kwargs={'tool_calls': [{'id': 'call_iF0vYWAd6rfely0cXSqdMOnF', 'function': {'arguments': '{"input":"3"}', 'name': 'magic_function'}, 'type': 'function'}]} response_metadata={'token_usage': {'completion_tokens': 31, 'prompt_tokens': 88, 'total_tokens': 119}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'tool_calls', 'logprobs': None} id='run-eb88ec77-d492-43a5-a5dd-4cefef9a6920-0' tool_calls=[{'name': 'magic_function', 'args': {'input': '3'}, 'id': 'call_iF0vYWAd6rfely0cXSqdMOnF', 'type': 'tool_call'}] usage_metadata={'input_tokens': 88, 'output_tokens': 31, 'total_tokens': 119}
content='Sorry, there was an error. Please try again.' name='magic_function' id='c9ff261f-a0f1-4c92-a9f2-cd749f62d911' tool_call_id='call_iF0vYWAd6rfely0cXSqdMOnF'
content='I am currently unable to process the request with the input "3" for the `magic_function`. If you have any other questions or need assistance with something else, please let me know!' response_metadata={'token_usage': {'completion_tokens': 39, 'prompt_tokens': 141, 'total_tokens': 180}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'stop', 'logprobs': None} id='run-d42508aa-f286-4b57-80fb-f8a76736d470-0' usage_metadata={'input_tokens': 141, 'output_tokens': 39, 'total_tokens': 180}
```

## `max_execution_time`

### 在 LangChain 中

`AgentExecutor` 实现了一个 `max_execution_time` 参数，允许用户中止超出总时间限制的运行。

```python
import time


@tool
def magic_function(input: str) -> str:
    """Applies a magic function to an input."""
    time.sleep(2.5)
    return "Sorry, there was an error. Please try again."


tools = [magic_function]

agent = create_tool_calling_agent(model, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    max_execution_time=2,
    verbose=True,
)

agent_executor.invoke({"input": query})
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Invoking: `magic_function` with `{'input': '3'}`


[0m[36;1m[1;3mSorry, there was an error. Please try again.[0m[32;1m[1;3m[0m

[1m> Finished chain.[0m
```


```output
{'input': 'what is the value of magic_function(3)?',
 'output': 'Agent stopped due to max iterations.'}
```

### 在 LangGraph 中

使用 LangGraph 的 react agent，您可以在两个层级上控制超时。

您可以设置 `step_timeout` 来限制每个 **step**：

```python
from langgraph.prebuilt import create_react_agent

app = create_react_agent(model, tools=tools)
# 在此处设置每个步骤的最大超时
app.step_timeout = 2

try:
    for chunk in app.stream({"messages": [("human", query)]}):
        print(chunk)
        print("------")
except TimeoutError:
    print({"input": query, "output": "代理因达到最大迭代次数而停止."})
```
```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_FKiTkTd0Ffd4rkYSzERprf1M', 'function': {'arguments': '{"input":"3"}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 55, 'total_tokens': 69}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-b842f7b6-ec10-40f8-8c0e-baa220b77e91-0', tool_calls=[{'name': 'magic_function', 'args': {'input': '3'}, 'id': 'call_FKiTkTd0Ffd4rkYSzERprf1M', 'type': 'tool_call'}], usage_metadata={'input_tokens': 55, 'output_tokens': 14, 'total_tokens': 69})]}}
------
{'input': 'magic_function(3) 的值是多少？', 'output': '代理因达到最大迭代次数而停止.'}
```
设置整个运行的单个最大超时的另一种方法是直接使用 Python 标准库 [asyncio](https://docs.python.org/3/library/asyncio.html)。

```python
import asyncio

from langgraph.prebuilt import create_react_agent

app = create_react_agent(model, tools=tools)


async def stream(app, inputs):
    async for chunk in app.astream({"messages": [("human", query)]}):
        print(chunk)
        print("------")


try:
    task = asyncio.create_task(stream(app, {"messages": [("human", query)]}))
    await asyncio.wait_for(task, timeout=3)
except TimeoutError:
    print("任务已取消.")
```
```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_WoOB8juagB08xrP38twYlYKR', 'function': {'arguments': '{"input":"3"}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 55, 'total_tokens': 69}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-73dee47e-30ab-42c9-bb0c-6f227cac96cd-0', tool_calls=[{'name': 'magic_function', 'args': {'input': '3'}, 'id': 'call_WoOB8juagB08xrP38twYlYKR', 'type': 'tool_call'}], usage_metadata={'input_tokens': 55, 'output_tokens': 14, 'total_tokens': 69})]}}
------
任务已取消.
```

## `early_stopping_method`

### 在 LangChain 中

使用 LangChain 的 [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.iter)，您可以配置一个 [early_stopping_method](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.early_stopping_method)，以返回一个字符串，说明“代理因迭代限制或时间限制而停止。”（`"force"`）或提示 LLM 最后一次响应（`"generate"`）。

```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o")


prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        ("human", "{input}"),
        # Placeholders fill up a **list** of messages
        ("placeholder", "{agent_scratchpad}"),
    ]
)


@tool
def magic_function(input: int) -> int:
    """Applies a magic function to an input."""
    return "Sorry there was an error, please try again."


tools = [magic_function]

agent = create_tool_calling_agent(model, tools, prompt=prompt)
agent_executor = AgentExecutor(
    agent=agent, tools=tools, early_stopping_method="force", max_iterations=1
)

result = agent_executor.invoke({"input": query})
print("Output with early_stopping_method='force':")
print(result["output"])
```
```output
Output with early_stopping_method='force':
Agent stopped due to max iterations.
```

### 在 LangGraph 中

在 LangGraph 中，您可以明确地处理代理之外的响应行为，因为可以访问完整的状态。

```python
from langgraph.errors import GraphRecursionError
from langgraph.prebuilt import create_react_agent

RECURSION_LIMIT = 2 * 1 + 1

app = create_react_agent(model, tools=tools)

try:
    for chunk in app.stream(
        {"messages": [("human", query)]},
        {"recursion_limit": RECURSION_LIMIT},
        stream_mode="values",
    ):
        print(chunk["messages"][-1])
except GraphRecursionError:
    print({"input": query, "output": "由于达到最大迭代次数，代理已停止."})
```
```output
content='what is the value of magic_function(3)?' id='4fa7fbe5-758c-47a3-9268-717665d10680'
content='' additional_kwargs={'tool_calls': [{'id': 'call_ujE0IQBbIQnxcF9gsZXQfdhF', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]} response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 55, 'total_tokens': 69}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'tool_calls', 'logprobs': None} id='run-65d689aa-baee-4342-a5d2-048feefab418-0' tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_ujE0IQBbIQnxcF9gsZXQfdhF', 'type': 'tool_call'}] usage_metadata={'input_tokens': 55, 'output_tokens': 14, 'total_tokens': 69}
content='Sorry there was an error, please try again.' name='magic_function' id='ef8ddf1d-9ad7-4ac0-b784-b673c4d94bbd' tool_call_id='call_ujE0IQBbIQnxcF9gsZXQfdhF'
content='It seems there was an issue with the previous attempt. Let me try that again.' additional_kwargs={'tool_calls': [{'id': 'call_GcsAfCFUHJ50BN2IOWnwTbQ7', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]} response_metadata={'token_usage': {'completion_tokens': 32, 'prompt_tokens': 87, 'total_tokens': 119}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'tool_calls', 'logprobs': None} id='run-54527c4b-8ff0-4ee8-8abf-224886bd222e-0' tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_GcsAfCFUHJ50BN2IOWnwTbQ7', 'type': 'tool_call'}] usage_metadata={'input_tokens': 87, 'output_tokens': 32, 'total_tokens': 119}
{'input': 'what is the value of magic_function(3)?', 'output': '由于达到最大迭代次数，代理已停止.'}
```

## `trim_intermediate_steps`

### 在 LangChain 中

使用 LangChain 的 [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor)，您可以通过 [trim_intermediate_steps](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.trim_intermediate_steps) 来减少长时间运行的代理的中间步骤，该参数可以是一个整数（表示代理应保留最后 N 步）或自定义函数。

例如，我们可以修剪该值，使代理仅查看最近的中间步骤。

```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o")


prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        ("human", "{input}"),
        # Placeholders fill up a **list** of messages
        ("placeholder", "{agent_scratchpad}"),
    ]
)


magic_step_num = 1


@tool
def magic_function(input: int) -> int:
    """Applies a magic function to an input."""
    global magic_step_num
    print(f"Call number: {magic_step_num}")
    magic_step_num += 1
    return input + magic_step_num


tools = [magic_function]

agent = create_tool_calling_agent(model, tools, prompt=prompt)


def trim_steps(steps: list):
    # Let's give the agent amnesia
    return []


agent_executor = AgentExecutor(
    agent=agent, tools=tools, trim_intermediate_steps=trim_steps
)


query = "Call the magic function 4 times in sequence with the value 3. You cannot call it multiple times at once."

for step in agent_executor.stream({"input": query}):
    pass
```
```output
Call number: 1
Call number: 2
Call number: 3
Call number: 4
Call number: 5
Call number: 6
Call number: 7
Call number: 8
Call number: 9
Call number: 10
Call number: 11
Call number: 12
Call number: 13
Call number: 14
``````output
Stopping agent prematurely due to triggering stop condition
``````output
Call number: 15
```

### 在 LangGraph 中

我们可以像之前一样使用 [`state_modifier`](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) 来传递 [prompt templates](#prompt-templates)。

```python
from langgraph.errors import GraphRecursionError
from langgraph.prebuilt import create_react_agent
from langgraph.prebuilt.chat_agent_executor import AgentState

magic_step_num = 1


@tool
def magic_function(input: int) -> int:
    """Applies a magic function to an input."""
    global magic_step_num
    print(f"Call number: {magic_step_num}")
    magic_step_num += 1
    return input + magic_step_num


tools = [magic_function]


def _modify_state_messages(state: AgentState):
    # Give the agent amnesia, only keeping the original user query
    return [("system", "You are a helpful assistant"), state["messages"][0]]


app = create_react_agent(model, tools, state_modifier=_modify_state_messages)

try:
    for step in app.stream({"messages": [("human", query)]}, stream_mode="updates"):
        pass
except GraphRecursionError as e:
    print("Stopping agent prematurely due to triggering stop condition")
```
```output
Call number: 1
Call number: 2
Call number: 3
Call number: 4
Call number: 5
Call number: 6
Call number: 7
Call number: 8
Call number: 9
Call number: 10
Call number: 11
Call number: 12
Stopping agent prematurely due to triggering stop condition
```

## 下一步

您现在已经学习了如何将您的 LangChain agent executors 迁移到 LangGraph。

接下来，请查看其他 [LangGraph 使用指南](https://langchain-ai.github.io/langgraph/how-tos/)。