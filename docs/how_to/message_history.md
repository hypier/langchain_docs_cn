---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/message_history.ipynb
keywords: [memory]
---
# How to add message history

:::info Prerequisites

This guide assumes familiarity with the following concepts:
- [LangChain Expression Language (LCEL)](/docs/concepts/#langchain-expression-language)
- [Chaining runnables](/docs/how_to/sequence/)
- [Configuring chain parameters at runtime](/docs/how_to/configure)
- [Prompt templates](/docs/concepts/#prompt-templates)
- [Chat Messages](/docs/concepts/#message-types)

:::

Passing conversation state into and out a chain is vital when building a chatbot. The [`RunnableWithMessageHistory`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.history.RunnableWithMessageHistory.html#langchain_core.runnables.history.RunnableWithMessageHistory) class lets us add message history to certain types of chains. It wraps another Runnable and manages the chat message history for it. Specifically, it loads previous messages in the conversation BEFORE passing it to the Runnable, and it saves the generated response as a message AFTER calling the runnable. This class also enables multiple conversations by saving each conversation with a `session_id` - it then expects a `session_id` to be passed in the config when calling the runnable, and uses that to look up the relevant conversation history.

![index_diagram](../../static/img/message_history.png)

In practice this looks something like:

```python
from langchain_core.runnables.history import RunnableWithMessageHistory


with_message_history = RunnableWithMessageHistory(
    # The underlying runnable
    runnable,  
    # A function that takes in a session id and returns a memory object
    get_session_history,  
    # Other parameters that may be needed to align the inputs/outputs
    # of the Runnable with the memory object
    ...  
)

with_message_history.invoke(
    # The same input as before
    {"ability": "math", "input": "What does cosine mean?"},
    # Configuration specifying the `session_id`,
    # which controls which conversation to load
    config={"configurable": {"session_id": "abc123"}},
)
```


In order to properly set this up there are two main things to consider:

1. How to store and load messages? (this is `get_session_history` in the example above)
2. What is the underlying Runnable you are wrapping and what are its inputs/outputs? (this is `runnable` in the example above, as well any additional parameters you pass to `RunnableWithMessageHistory` to align the inputs/outputs)

Let's walk through these pieces (and more) below.

## How to store and load messages

A key part of this is storing and loading messages.
When constructing `RunnableWithMessageHistory` you need to pass in a `get_session_history` function.
This function should take in a `session_id` and return a `BaseChatMessageHistory` object.

**What is `session_id`?** 

`session_id` is an identifier for the session (conversation) thread that these input messages correspond to. This allows you to maintain several conversations/threads with the same chain at the same time.

**What is `BaseChatMessageHistory`?** 

`BaseChatMessageHistory` is a class that can load and save message objects. It will be called by `RunnableWithMessageHistory` to do exactly that. These classes are usually initialized with a session id.

Let's create a `get_session_history` object to use for this example. To keep things simple, we will use a simple SQLiteMessage


```python
! rm memory.db
```


```python
from langchain_community.chat_message_histories import SQLChatMessageHistory


def get_session_history(session_id):
    return SQLChatMessageHistory(session_id, "sqlite:///memory.db")
```

Check out the [memory integrations](https://integrations.langchain.com/memory) page for implementations of chat message histories using other providers (Redis, Postgres, etc).

## What is the runnable you are trying to wrap?

`RunnableWithMessageHistory` can only wrap certain types of Runnables. Specifically, it can be used for any Runnable that takes as input one of:

* a sequence of [`BaseMessages`](/docs/concepts/#message-types)
* a dict with a key that takes a sequence of `BaseMessages`
* a dict with a key that takes the latest message(s) as a string or sequence of `BaseMessages`, and a separate key that takes historical messages

And returns as output one of

* a string that can be treated as the contents of an `AIMessage`
* a sequence of `BaseMessage`
* a dict with a key that contains a sequence of `BaseMessage`

Let's take a look at some examples to see how it works. 

### Setup

First we construct a runnable (which here accepts a dict as input and returns a message as output):

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs
  customVarName="llm"
/>


```python
from langchain_core.messages import HumanMessage
from langchain_core.runnables.history import RunnableWithMessageHistory
```

### Messages input, message(s) output

The simplest form is just adding memory to a ChatModel.
ChatModels accept a list of messages as input and output a message.
This makes it very easy to use `RunnableWithMessageHistory` - no additional configuration is needed!


```python
runnable_with_history = RunnableWithMessageHistory(
    model,
    get_session_history,
)
```


```python
runnable_with_history.invoke(
    [HumanMessage(content="hi - im bob!")],
    config={"configurable": {"session_id": "1"}},
)
```



```output
AIMessage(content="It's nice to meet you, Bob! I'm Claude, an AI assistant created by Anthropic. How can I help you today?", response_metadata={'id': 'msg_01UHCCMiZz9yNYjt41xUJrtk', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 12, 'output_tokens': 32}}, id='run-55f6a451-606b-4e04-9e39-e03b81035c1f-0', usage_metadata={'input_tokens': 12, 'output_tokens': 32, 'total_tokens': 44})
```



```python
runnable_with_history.invoke(
    [HumanMessage(content="whats my name?")],
    config={"configurable": {"session_id": "1"}},
)
```



```output
AIMessage(content='I\'m afraid I don\'t actually know your name - you introduced yourself as Bob, but I don\'t have any other information about your identity. As an AI assistant, I don\'t have a way to independently verify people\'s names or identities. I\'m happy to continue our conversation, but I\'ll just refer to you as "Bob" since that\'s the name you provided.', response_metadata={'id': 'msg_018L96tAxiexMKsHBQz22CcE', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 52, 'output_tokens': 80}}, id='run-7399ddb5-bb06-444b-bfb2-2f65674105dd-0', usage_metadata={'input_tokens': 52, 'output_tokens': 80, 'total_tokens': 132})
```


:::info

Note that in this case the context is preserved via the chat history for the provided `session_id`, so the model knows the users name.

:::

We can now try this with a new session id and see that it does not remember.


```python
runnable_with_history.invoke(
    [HumanMessage(content="whats my name?")],
    config={"configurable": {"session_id": "1a"}},
)
```



```output
AIMessage(content="I'm afraid I don't actually know your name. As an AI assistant, I don't have personal information about you unless you provide it to me directly.", response_metadata={'id': 'msg_01LhbWu7mSKTvKAx7iQpMPzd', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 12, 'output_tokens': 35}}, id='run-cf86cad2-21f2-4525-afc8-09bfd1e8af70-0', usage_metadata={'input_tokens': 12, 'output_tokens': 35, 'total_tokens': 47})
```


:::info      

When we pass a different `session_id`, we start a new chat history, so the model does not know what the user's name is. 

:::

### Dictionary input, message(s) output

Besides just wrapping a raw model, the next step up is wrapping a prompt + LLM. This now changes the input to be a **dictionary** (because the input to a prompt is a dictionary). This adds two bits of complication.

First: a dictionary can have multiple keys, but we only want to save ONE as input. In order to do this, we now now need to specify a key to save as the input.

Second: once we load the messages, we need to know how to save them to the dictionary. That equates to know which key in the dictionary to save them in. Therefore, we need to specify a key to save the loaded messages in.

Putting it all together, that ends up looking something like:


```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You're an assistant who speaks in {language}. Respond in 20 words or fewer",
        ),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{input}"),
    ]
)

runnable = prompt | model

runnable_with_history = RunnableWithMessageHistory(
    runnable,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history",
)
```

:::info

Note that we've specified `input_messages_key` (the key to be treated as the latest input message) and `history_messages_key` (the key to add historical messages to).

:::


```python
runnable_with_history.invoke(
    {"language": "italian", "input": "hi im bob!"},
    config={"configurable": {"session_id": "2"}},
)
```



```output
AIMessage(content='Ciao Bob! È un piacere conoscerti. Come stai oggi?', response_metadata={'id': 'msg_0121ADUEe4G1hMC6zbqFWofr', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 29, 'output_tokens': 23}}, id='run-246a70df-aad6-43d6-a7e8-166d96e0d67e-0', usage_metadata={'input_tokens': 29, 'output_tokens': 23, 'total_tokens': 52})
```



```python
runnable_with_history.invoke(
    {"language": "italian", "input": "whats my name?"},
    config={"configurable": {"session_id": "2"}},
)
```



```output
AIMessage(content='Bob, il tuo nome è Bob.', response_metadata={'id': 'msg_01EDUZG6nRLGeti9KhFN5cek', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 60, 'output_tokens': 12}}, id='run-294b4a72-81bc-4c43-b199-3aafdff87cb3-0', usage_metadata={'input_tokens': 60, 'output_tokens': 12, 'total_tokens': 72})
```


:::info

Note that in this case the context is preserved via the chat history for the provided `session_id`, so the model knows the users name.

:::

We can now try this with a new session id and see that it does not remember.


```python
runnable_with_history.invoke(
    {"language": "italian", "input": "whats my name?"},
    config={"configurable": {"session_id": "2a"}},
)
```



```output
AIMessage(content='Mi dispiace, non so il tuo nome. Come posso aiutarti?', response_metadata={'id': 'msg_01Lyd9FAGQJTxxAZoFi3sQpQ', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 30, 'output_tokens': 23}}, id='run-19a82197-3b1c-4b5f-a68d-f91f4a2ba523-0', usage_metadata={'input_tokens': 30, 'output_tokens': 23, 'total_tokens': 53})
```


:::info      

When we pass a different `session_id`, we start a new chat history, so the model does not know what the user's name is. 

:::

### Messages input, dict output

This format is useful when you are using a model to generate one key in a dictionary.


```python
from langchain_core.messages import HumanMessage
from langchain_core.runnables import RunnableParallel

chain = RunnableParallel({"output_message": model})


runnable_with_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    output_messages_key="output_message",
)
```

:::info

Note that we've specified `output_messages_key` (the key to be treated as the output to save).

:::


```python
runnable_with_history.invoke(
    [HumanMessage(content="hi - im bob!")],
    config={"configurable": {"session_id": "3"}},
)
```



```output
{'output_message': AIMessage(content="It's nice to meet you, Bob! I'm Claude, an AI assistant created by Anthropic. How can I help you today?", response_metadata={'id': 'msg_01WWJSyUyGGKuBqTs3h18ZMM', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 12, 'output_tokens': 32}}, id='run-0f50cb43-a734-447c-b535-07c615a0984c-0', usage_metadata={'input_tokens': 12, 'output_tokens': 32, 'total_tokens': 44})}
```



```python
runnable_with_history.invoke(
    [HumanMessage(content="whats my name?")],
    config={"configurable": {"session_id": "3"}},
)
```



```output
{'output_message': AIMessage(content='I\'m afraid I don\'t actually know your name - you introduced yourself as Bob, but I don\'t have any other information about your identity. As an AI assistant, I don\'t have a way to independently verify people\'s names or identities. I\'m happy to continue our conversation, but I\'ll just refer to you as "Bob" since that\'s the name you provided.', response_metadata={'id': 'msg_01TEGrhfLXTwo36rC7svdTy4', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 52, 'output_tokens': 80}}, id='run-178e8f3f-da21-430d-9edc-ef07797a5e2d-0', usage_metadata={'input_tokens': 52, 'output_tokens': 80, 'total_tokens': 132})}
```


:::info

Note that in this case the context is preserved via the chat history for the provided `session_id`, so the model knows the users name.

:::

We can now try this with a new session id and see that it does not remember.


```python
runnable_with_history.invoke(
    [HumanMessage(content="whats my name?")],
    config={"configurable": {"session_id": "3a"}},
)
```



```output
{'output_message': AIMessage(content="I'm afraid I don't actually know your name. As an AI assistant, I don't have personal information about you unless you provide it to me directly.", response_metadata={'id': 'msg_0118ZBudDXAC9P6smf91NhCX', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 12, 'output_tokens': 35}}, id='run-deb14a3a-0336-42b4-8ace-ad1e52ca5910-0', usage_metadata={'input_tokens': 12, 'output_tokens': 35, 'total_tokens': 47})}
```


:::info      

When we pass a different `session_id`, we start a new chat history, so the model does not know what the user's name is. 

:::

### Dict with single key for all messages input, messages output

This is a specific case of "Dictionary input, message(s) output". In this situation, because there is only a single key we don't need to specify as much - we only need to specify the `input_messages_key`.


```python
from operator import itemgetter

runnable_with_history = RunnableWithMessageHistory(
    itemgetter("input_messages") | model,
    get_session_history,
    input_messages_key="input_messages",
)
```

:::info

Note that we've specified `input_messages_key` (the key to be treated as the latest input message).

:::


```python
runnable_with_history.invoke(
    {"input_messages": [HumanMessage(content="hi - im bob!")]},
    config={"configurable": {"session_id": "4"}},
)
```



```output
AIMessage(content="It's nice to meet you, Bob! I'm Claude, an AI assistant created by Anthropic. How can I help you today?", response_metadata={'id': 'msg_01UdD5wz1J5xwoz5D94onaQC', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 12, 'output_tokens': 32}}, id='run-91bee6eb-0814-4557-ad71-fef9b0270358-0', usage_metadata={'input_tokens': 12, 'output_tokens': 32, 'total_tokens': 44})
```



```python
runnable_with_history.invoke(
    {"input_messages": [HumanMessage(content="whats my name?")]},
    config={"configurable": {"session_id": "4"}},
)
```



```output
AIMessage(content='I\'m afraid I don\'t actually know your name - you introduced yourself as Bob, but I don\'t have any other information about your identity. As an AI assistant, I don\'t have a way to independently verify people\'s names or identities. I\'m happy to continue our conversation, but I\'ll just refer to you as "Bob" since that\'s the name you provided.', response_metadata={'id': 'msg_012WUygxBKXcVJPeTW14LNrc', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 52, 'output_tokens': 80}}, id='run-fcbaaa1a-8c33-4eec-b0b0-5b800a47bddd-0', usage_metadata={'input_tokens': 52, 'output_tokens': 80, 'total_tokens': 132})
```


:::info

Note that in this case the context is preserved via the chat history for the provided `session_id`, so the model knows the users name.

:::

We can now try this with a new session id and see that it does not remember.


```python
runnable_with_history.invoke(
    {"input_messages": [HumanMessage(content="whats my name?")]},
    config={"configurable": {"session_id": "4a"}},
)
```



```output
AIMessage(content="I'm afraid I don't actually know your name. As an AI assistant, I don't have personal information about you unless you provide it to me directly.", response_metadata={'id': 'msg_017xW3Ki5y4UBYzCU9Mf1pgM', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 12, 'output_tokens': 35}}, id='run-d2f372f7-3679-4a5c-9331-a55b820ec03e-0', usage_metadata={'input_tokens': 12, 'output_tokens': 35, 'total_tokens': 47})
```


:::info      

When we pass a different `session_id`, we start a new chat history, so the model does not know what the user's name is. 

:::

## Customization

The configuration parameters by which we track message histories can be customized by passing in a list of ``ConfigurableFieldSpec`` objects to the ``history_factory_config`` parameter. Below, we use two parameters: a `user_id` and `conversation_id`.


```python
from langchain_core.runnables import ConfigurableFieldSpec


def get_session_history(user_id: str, conversation_id: str):
    return SQLChatMessageHistory(f"{user_id}--{conversation_id}", "sqlite:///memory.db")


with_message_history = RunnableWithMessageHistory(
    runnable,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history",
    history_factory_config=[
        ConfigurableFieldSpec(
            id="user_id",
            annotation=str,
            name="User ID",
            description="Unique identifier for the user.",
            default="",
            is_shared=True,
        ),
        ConfigurableFieldSpec(
            id="conversation_id",
            annotation=str,
            name="Conversation ID",
            description="Unique identifier for the conversation.",
            default="",
            is_shared=True,
        ),
    ],
)

with_message_history.invoke(
    {"language": "italian", "input": "hi im bob!"},
    config={"configurable": {"user_id": "123", "conversation_id": "1"}},
)
```



```output
AIMessage(content='Ciao Bob! È un piacere conoscerti. Come stai oggi?', response_metadata={'id': 'msg_016RJebCoiAgWaNcbv9wrMNW', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 29, 'output_tokens': 23}}, id='run-40425414-8f72-47d4-bf1d-a84175d8b3f8-0', usage_metadata={'input_tokens': 29, 'output_tokens': 23, 'total_tokens': 52})
```



```python
# remembers
with_message_history.invoke(
    {"language": "italian", "input": "whats my name?"},
    config={"configurable": {"user_id": "123", "conversation_id": "1"}},
)
```



```output
AIMessage(content='Bob, il tuo nome è Bob.', response_metadata={'id': 'msg_01Kktiy3auFDKESY54KtTWPX', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 60, 'output_tokens': 12}}, id='run-c7768420-3f30-43f5-8834-74b1979630dd-0', usage_metadata={'input_tokens': 60, 'output_tokens': 12, 'total_tokens': 72})
```



```python
# New user_id --> does not remember
with_message_history.invoke(
    {"language": "italian", "input": "whats my name?"},
    config={"configurable": {"user_id": "456", "conversation_id": "1"}},
)
```



```output
AIMessage(content='Mi dispiace, non so il tuo nome. Come posso aiutarti?', response_metadata={'id': 'msg_0178FpbpPNioB7kqvyHk7rjD', 'model': 'claude-3-haiku-20240307', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 30, 'output_tokens': 23}}, id='run-df1f1768-aab6-4aec-8bba-e33fc9e90b8d-0', usage_metadata={'input_tokens': 30, 'output_tokens': 23, 'total_tokens': 53})
```


Note that in this case the context was preserved for the same `user_id`, but once we changed it, the new chat history was started, even though the `conversation_id` was the same.
