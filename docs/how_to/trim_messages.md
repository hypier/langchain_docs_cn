---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/trim_messages.ipynb
---
# How to trim messages

:::info Prerequisites

This guide assumes familiarity with the following concepts:

- [Messages](/docs/concepts/#messages)
- [Chat models](/docs/concepts/#chat-models)
- [Chaining](/docs/how_to/sequence/)
- [Chat history](/docs/concepts/#chat-history)

The methods in this guide also require `langchain-core>=0.2.9`.

:::

All models have finite context windows, meaning there's a limit to how many tokens they can take as input. If you have very long messages or a chain/agent that accumulates a long message is history, you'll need to manage the length of the messages you're passing in to the model.

The `trim_messages` util provides some basic strategies for trimming a list of messages to be of a certain token length.

## Getting the last `max_tokens` tokens

To get the last `max_tokens` in the list of Messages we can set `strategy="last"`. Notice that for our `token_counter` we can pass in a function (more on that below) or a language model (since language models have a message token counting method). It makes sense to pass in a model when you're trimming your messages to fit into the context window of that specific model:


```python
# pip install -U langchain-openai
from langchain_core.messages import (
    AIMessage,
    HumanMessage,
    SystemMessage,
    trim_messages,
)
from langchain_openai import ChatOpenAI

messages = [
    SystemMessage("you're a good assistant, you always respond with a joke."),
    HumanMessage("i wonder why it's called langchain"),
    AIMessage(
        'Well, I guess they thought "WordRope" and "SentenceString" just didn\'t have the same ring to it!'
    ),
    HumanMessage("and who is harrison chasing anyways"),
    AIMessage(
        "Hmmm let me think.\n\nWhy, he's probably chasing after the last cup of coffee in the office!"
    ),
    HumanMessage("what do you call a speechless parrot"),
]

trim_messages(
    messages,
    max_tokens=45,
    strategy="last",
    token_counter=ChatOpenAI(model="gpt-4o"),
)
```



```output
[AIMessage(content="Hmmm let me think.\n\nWhy, he's probably chasing after the last cup of coffee in the office!"),
 HumanMessage(content='what do you call a speechless parrot')]
```


If we want to always keep the initial system message we can specify `include_system=True`:


```python
trim_messages(
    messages,
    max_tokens=45,
    strategy="last",
    token_counter=ChatOpenAI(model="gpt-4o"),
    include_system=True,
)
```



```output
[SystemMessage(content="you're a good assistant, you always respond with a joke."),
 HumanMessage(content='what do you call a speechless parrot')]
```


If we want to allow splitting up the contents of a message we can specify `allow_partial=True`:


```python
trim_messages(
    messages,
    max_tokens=56,
    strategy="last",
    token_counter=ChatOpenAI(model="gpt-4o"),
    include_system=True,
    allow_partial=True,
)
```



```output
[SystemMessage(content="you're a good assistant, you always respond with a joke."),
 AIMessage(content="\nWhy, he's probably chasing after the last cup of coffee in the office!"),
 HumanMessage(content='what do you call a speechless parrot')]
```


If we need to make sure that our first message (excluding the system message) is always of a specific type, we can specify `start_on`:


```python
trim_messages(
    messages,
    max_tokens=60,
    strategy="last",
    token_counter=ChatOpenAI(model="gpt-4o"),
    include_system=True,
    start_on="human",
)
```



```output
[SystemMessage(content="you're a good assistant, you always respond with a joke."),
 HumanMessage(content='what do you call a speechless parrot')]
```


## Getting the first `max_tokens` tokens

We can perform the flipped operation of getting the *first* `max_tokens` by specifying `strategy="first"`:


```python
trim_messages(
    messages,
    max_tokens=45,
    strategy="first",
    token_counter=ChatOpenAI(model="gpt-4o"),
)
```



```output
[SystemMessage(content="you're a good assistant, you always respond with a joke."),
 HumanMessage(content="i wonder why it's called langchain")]
```


## Writing a custom token counter

We can write a custom token counter function that takes in a list of messages and returns an int.


```python
from typing import List

# pip install tiktoken
import tiktoken
from langchain_core.messages import BaseMessage, ToolMessage


def str_token_counter(text: str) -> int:
    enc = tiktoken.get_encoding("o200k_base")
    return len(enc.encode(text))


def tiktoken_counter(messages: List[BaseMessage]) -> int:
    """Approximately reproduce https://github.com/openai/openai-cookbook/blob/main/examples/How_to_count_tokens_with_tiktoken.ipynb

    For simplicity only supports str Message.contents.
    """
    num_tokens = 3  # every reply is primed with <|start|>assistant<|message|>
    tokens_per_message = 3
    tokens_per_name = 1
    for msg in messages:
        if isinstance(msg, HumanMessage):
            role = "user"
        elif isinstance(msg, AIMessage):
            role = "assistant"
        elif isinstance(msg, ToolMessage):
            role = "tool"
        elif isinstance(msg, SystemMessage):
            role = "system"
        else:
            raise ValueError(f"Unsupported messages type {msg.__class__}")
        num_tokens += (
            tokens_per_message
            + str_token_counter(role)
            + str_token_counter(msg.content)
        )
        if msg.name:
            num_tokens += tokens_per_name + str_token_counter(msg.name)
    return num_tokens


trim_messages(
    messages,
    max_tokens=45,
    strategy="last",
    token_counter=tiktoken_counter,
)
```



```output
[AIMessage(content="Hmmm let me think.\n\nWhy, he's probably chasing after the last cup of coffee in the office!"),
 HumanMessage(content='what do you call a speechless parrot')]
```


## Chaining

`trim_messages` can be used in an imperatively (like above) or declaratively, making it easy to compose with other components in a chain


```python
llm = ChatOpenAI(model="gpt-4o")

# Notice we don't pass in messages. This creates
# a RunnableLambda that takes messages as input
trimmer = trim_messages(
    max_tokens=45,
    strategy="last",
    token_counter=llm,
    include_system=True,
)

chain = trimmer | llm
chain.invoke(messages)
```



```output
AIMessage(content='A: A "Polly-gone"!', response_metadata={'token_usage': {'completion_tokens': 9, 'prompt_tokens': 32, 'total_tokens': 41}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_66b29dffce', 'finish_reason': 'stop', 'logprobs': None}, id='run-83e96ddf-bcaa-4f63-824c-98b0f8a0d474-0', usage_metadata={'input_tokens': 32, 'output_tokens': 9, 'total_tokens': 41})
```


Looking at the LangSmith trace we can see that before the messages are passed to the model they are first trimmed: https://smith.langchain.com/public/65af12c4-c24d-4824-90f0-6547566e59bb/r

Looking at just the trimmer, we can see that it's a Runnable object that can be invoked like all Runnables:


```python
trimmer.invoke(messages)
```



```output
[SystemMessage(content="you're a good assistant, you always respond with a joke."),
 HumanMessage(content='what do you call a speechless parrot')]
```


## Using with ChatMessageHistory

Trimming messages is especially useful when [working with chat histories](/docs/how_to/message_history/), which can get arbitrarily long:


```python
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

chat_history = InMemoryChatMessageHistory(messages=messages[:-1])


def dummy_get_session_history(session_id):
    if session_id != "1":
        return InMemoryChatMessageHistory()
    return chat_history


llm = ChatOpenAI(model="gpt-4o")

trimmer = trim_messages(
    max_tokens=45,
    strategy="last",
    token_counter=llm,
    include_system=True,
)

chain = trimmer | llm
chain_with_history = RunnableWithMessageHistory(chain, dummy_get_session_history)
chain_with_history.invoke(
    [HumanMessage("what do you call a speechless parrot")],
    config={"configurable": {"session_id": "1"}},
)
```



```output
AIMessage(content='A "polly-no-wanna-cracker"!', response_metadata={'token_usage': {'completion_tokens': 10, 'prompt_tokens': 32, 'total_tokens': 42}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_5bf7397cd3', 'finish_reason': 'stop', 'logprobs': None}, id='run-054dd309-3497-4e7b-b22a-c1859f11d32e-0', usage_metadata={'input_tokens': 32, 'output_tokens': 10, 'total_tokens': 42})
```


Looking at the LangSmith trace we can see that we retrieve all of our messages but before the messages are passed to the model they are trimmed to be just the system message and last human message: https://smith.langchain.com/public/17dd700b-9994-44ca-930c-116e00997315/r

## API reference

For a complete description of all arguments head to the API reference: https://api.python.langchain.com/en/latest/messages/langchain_core.messages.utils.trim_messages.html
