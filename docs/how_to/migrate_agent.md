---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/migrate_agent.ipynb
keywords: [create_react_agent, create_react_agent()]
---
# How to migrate from legacy LangChain agents to LangGraph

:::info Prerequisites

This guide assumes familiarity with the following concepts:
- [Agents](/docs/concepts/#agents)
- [LangGraph](https://langchain-ai.github.io/langgraph/)
- [Tool calling](/docs/how_to/tool_calling/)

:::

Here we focus on how to move from legacy LangChain agents to more flexible [LangGraph](https://langchain-ai.github.io/langgraph/) agents.
LangChain agents (the [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor) in particular) have multiple configuration parameters.
In this notebook we will show how those parameters map to the LangGraph react agent executor using the [create_react_agent](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) prebuilt helper method.

#### Prerequisites

This how-to guide uses OpenAI as the LLM. Install the dependencies to run.


```python
%%capture --no-stderr
%pip install -U langgraph langchain langchain-openai
```

Then, set your OpenAI API key.


```python
import os

os.environ["OPENAI_API_KEY"] = "sk-..."
```

## Basic Usage

For basic creation and usage of a tool-calling ReAct-style agent, the functionality is the same. First, let's define a model and tool(s), then we'll use those to create an agent.


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

For the LangChain [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor), we define a prompt with a placeholder for the agent's scratchpad. The agent can be invoked as follows:


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


LangGraph's [react agent executor](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) manages a state that is defined by a list of messages. It will continue to process the list until there are no tool calls in the agent's output. To kick it off, we input a list of messages. The output will contain the entire state of the graph-- in this case, the conversation history.




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


## Prompt Templates

With legacy LangChain agents you have to pass in a prompt template. You can use this to control the agent.

With LangGraph [react agent executor](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent), by default there is no prompt. You can achieve similar control over the agent in a few ways:

1. Pass in a system message as input
2. Initialize the agent with a system message
3. Initialize the agent with a function to transform messages before passing to the model.

Let's take a look at all of these below. We will pass in custom instructions to get the agent to respond in Spanish.

First up, using `AgentExecutor`:


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


Now, let's pass a custom system message to [react agent executor](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent).

LangGraph's prebuilt `create_react_agent` does not take a prompt template directly as a parameter, but instead takes a [`state_modifier`](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) parameter. This modifies the graph state before the llm is called, and can be one of four values:

- A `SystemMessage`, which is added to the beginning of the list of messages.
- A `string`, which is converted to a `SystemMessage` and added to the beginning of the list of messages.
- A `Callable`, which should take in full graph state. The output is then passed to the language model.
- Or a [`Runnable`](/docs/concepts/#langchain-expression-language-lcel), which should take in full graph state. The output is then passed to the language model.

Here's how it looks in action:


```python
from langchain_core.messages import SystemMessage
from langgraph.prebuilt import create_react_agent

system_message = "You are a helpful assistant. Respond only in Spanish."
# This could also be a SystemMessage object
# system_message = SystemMessage(content="You are a helpful assistant. Respond only in Spanish.")

app = create_react_agent(model, tools, state_modifier=system_message)


messages = app.invoke({"messages": [("user", query)]})
```

We can also pass in an arbitrary function. This function should take in a list of messages and output a list of messages.
We can do all types of arbitrary formatting of messages here. In this cases, let's just add a SystemMessage to the start of the list of messages.


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
## Memory

### In LangChain

With LangChain's [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.iter), you could add chat [Memory](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.memory) so it can engage in a multi-turn conversation.


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
### In LangGraph

Memory is just [persistence](https://langchain-ai.github.io/langgraph/how-tos/persistence/), aka [checkpointing](https://langchain-ai.github.io/langgraph/reference/checkpoints/).

Add a `checkpointer` to the agent and you get chat memory for free.


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
## Iterating through steps

### In LangChain

With LangChain's [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.iter), you could iterate over the steps using the [stream](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.stream) (or async `astream`) methods or the [iter](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.iter) method. LangGraph supports stepwise iteration using [stream](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.stream) 


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
{'output': 'The value of `magic_function(3)` is 5.', 'messages': [AIMessage(content='The value of `magic_function(3)` is 5.')]}
```
### In LangGraph

In LangGraph, things are handled natively using [stream](https://langchain-ai.github.io/langgraph/reference/graphs/#langgraph.graph.graph.CompiledGraph.stream) or the asynchronous `astream` method.


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

### In LangChain

Setting this parameter on AgentExecutor allows users to access intermediate_steps, which pairs agent actions (e.g., tool invocations) with their outcomes.



```python
agent_executor = AgentExecutor(agent=agent, tools=tools, return_intermediate_steps=True)
result = agent_executor.invoke({"input": query})
print(result["intermediate_steps"])
```
```output
[(ToolAgentAction(tool='magic_function', tool_input={'input': 3}, log="\nInvoking: `magic_function` with `{'input': 3}`\n\n\n", message_log=[AIMessageChunk(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_uPZ2D1Bo5mdED3gwgaeWURrf', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_calls', 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518'}, id='run-a792db4a-278d-4090-82ae-904a30eada93', tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_uPZ2D1Bo5mdED3gwgaeWURrf', 'type': 'tool_call'}], tool_call_chunks=[{'name': 'magic_function', 'args': '{"input":3}', 'id': 'call_uPZ2D1Bo5mdED3gwgaeWURrf', 'index': 0, 'type': 'tool_call_chunk'}])], tool_call_id='call_uPZ2D1Bo5mdED3gwgaeWURrf'), 5)]
```
### In LangGraph

By default the [react agent executor](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) in LangGraph appends all messages to the central state. Therefore, it is easy to see any intermediate steps by just looking at the full state.


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

### In LangChain

`AgentExecutor` implements a `max_iterations` parameter, allowing users to abort a run that exceeds a specified number of iterations.


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


### In LangGraph

In LangGraph this is controlled via `recursion_limit` configuration parameter.

Note that in `AgentExecutor`, an "iteration" includes a full turn of tool invocation and execution. In LangGraph, each step contributes to the recursion limit, so we will need to multiply by two (and add one) to get equivalent results.

If the recursion limit is reached, LangGraph raises a specific exception type, that we can catch and manage similarly to AgentExecutor.


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

### In LangChain

`AgentExecutor` implements a `max_execution_time` parameter, allowing users to abort a run that exceeds a total time limit.


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


### In LangGraph

With LangGraph's react agent, you can control timeouts on two levels. 

You can set a `step_timeout` to bound each **step**:


```python
from langgraph.prebuilt import create_react_agent

app = create_react_agent(model, tools=tools)
# Set the max timeout for each step here
app.step_timeout = 2

try:
    for chunk in app.stream({"messages": [("human", query)]}):
        print(chunk)
        print("------")
except TimeoutError:
    print({"input": query, "output": "Agent stopped due to max iterations."})
```
```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_FKiTkTd0Ffd4rkYSzERprf1M', 'function': {'arguments': '{"input":"3"}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 55, 'total_tokens': 69}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-b842f7b6-ec10-40f8-8c0e-baa220b77e91-0', tool_calls=[{'name': 'magic_function', 'args': {'input': '3'}, 'id': 'call_FKiTkTd0Ffd4rkYSzERprf1M', 'type': 'tool_call'}], usage_metadata={'input_tokens': 55, 'output_tokens': 14, 'total_tokens': 69})]}}
------
{'input': 'what is the value of magic_function(3)?', 'output': 'Agent stopped due to max iterations.'}
```
The other way to set a single max timeout for an entire run is to directly use the python stdlib [asyncio](https://docs.python.org/3/library/asyncio.html) library.


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
    print("Task Cancelled.")
```
```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_WoOB8juagB08xrP38twYlYKR', 'function': {'arguments': '{"input":"3"}', 'name': 'magic_function'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 55, 'total_tokens': 69}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-73dee47e-30ab-42c9-bb0c-6f227cac96cd-0', tool_calls=[{'name': 'magic_function', 'args': {'input': '3'}, 'id': 'call_WoOB8juagB08xrP38twYlYKR', 'type': 'tool_call'}], usage_metadata={'input_tokens': 55, 'output_tokens': 14, 'total_tokens': 69})]}}
------
Task Cancelled.
```
## `early_stopping_method`

### In LangChain

With LangChain's [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.iter), you could configure an [early_stopping_method](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.early_stopping_method) to either return a string saying "Agent stopped due to iteration limit or time limit." (`"force"`) or prompt the LLM a final time to respond (`"generate"`).


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
### In LangGraph

In LangGraph, you can explicitly handle the response behavior outside the agent, since the full state can be accessed.


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
    print({"input": query, "output": "Agent stopped due to max iterations."})
```
```output
content='what is the value of magic_function(3)?' id='4fa7fbe5-758c-47a3-9268-717665d10680'
content='' additional_kwargs={'tool_calls': [{'id': 'call_ujE0IQBbIQnxcF9gsZXQfdhF', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]} response_metadata={'token_usage': {'completion_tokens': 14, 'prompt_tokens': 55, 'total_tokens': 69}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'tool_calls', 'logprobs': None} id='run-65d689aa-baee-4342-a5d2-048feefab418-0' tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_ujE0IQBbIQnxcF9gsZXQfdhF', 'type': 'tool_call'}] usage_metadata={'input_tokens': 55, 'output_tokens': 14, 'total_tokens': 69}
content='Sorry there was an error, please try again.' name='magic_function' id='ef8ddf1d-9ad7-4ac0-b784-b673c4d94bbd' tool_call_id='call_ujE0IQBbIQnxcF9gsZXQfdhF'
content='It seems there was an issue with the previous attempt. Let me try that again.' additional_kwargs={'tool_calls': [{'id': 'call_GcsAfCFUHJ50BN2IOWnwTbQ7', 'function': {'arguments': '{"input":3}', 'name': 'magic_function'}, 'type': 'function'}]} response_metadata={'token_usage': {'completion_tokens': 32, 'prompt_tokens': 87, 'total_tokens': 119}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_4e2b2da518', 'finish_reason': 'tool_calls', 'logprobs': None} id='run-54527c4b-8ff0-4ee8-8abf-224886bd222e-0' tool_calls=[{'name': 'magic_function', 'args': {'input': 3}, 'id': 'call_GcsAfCFUHJ50BN2IOWnwTbQ7', 'type': 'tool_call'}] usage_metadata={'input_tokens': 87, 'output_tokens': 32, 'total_tokens': 119}
{'input': 'what is the value of magic_function(3)?', 'output': 'Agent stopped due to max iterations.'}
```
## `trim_intermediate_steps`

### In LangChain

With LangChain's [AgentExecutor](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor), you could trim the intermediate steps of long-running agents using [trim_intermediate_steps](https://api.python.langchain.com/en/latest/agents/langchain.agents.agent.AgentExecutor.html#langchain.agents.agent.AgentExecutor.trim_intermediate_steps), which is either an integer (indicating the agent should keep the last N steps) or a custom function.

For instance, we could trim the value so the agent only sees the most recent intermediate step.


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
### In LangGraph

We can use the [`state_modifier`](https://langchain-ai.github.io/langgraph/reference/prebuilt/#create_react_agent) just as before when passing in [prompt templates](#prompt-templates).


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
## Next steps

You've now learned how to migrate your LangChain agent executors to LangGraph.

Next, check out other [LangGraph how-to guides](https://langchain-ai.github.io/langgraph/how-tos/).
