---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/tutorials/chatbot.ipynb
sidebar_position: 1
keywords: [conversationchain]
---
# Build a Chatbot

:::info Prerequisites

This guide assumes familiarity with the following concepts:

- [Chat Models](/docs/concepts/#chat-models)
- [Prompt Templates](/docs/concepts/#prompt-templates)
- [Chat History](/docs/concepts/#chat-history)

:::

## Overview

We'll go over an example of how to design and implement an LLM-powered chatbot. 
This chatbot will be able to have a conversation and remember previous interactions.


Note that this chatbot that we build will only use the language model to have a conversation.
There are several other related concepts that you may be looking for:

- [Conversational RAG](/docs/tutorials/qa_chat_history): Enable a chatbot experience over an external source of data
- [Agents](/docs/tutorials/agents): Build a chatbot that can take actions

This tutorial will cover the basics which will be helpful for those two more advanced topics, but feel free to skip directly to there should you choose.

## Setup

### Jupyter Notebook

This guide (and most of the other guides in the documentation) uses [Jupyter notebooks](https://jupyter.org/) and assumes the reader is as well. Jupyter notebooks are perfect for learning how to work with LLM systems because oftentimes things can go wrong (unexpected output, API down, etc) and going through guides in an interactive environment is a great way to better understand them.

This and other tutorials are perhaps most conveniently run in a Jupyter notebook. See [here](https://jupyter.org/install) for instructions on how to install.

### Installation

To install LangChain run:

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CodeBlock from "@theme/CodeBlock";

<Tabs>
  <TabItem value="pip" label="Pip" default>
    <CodeBlock language="bash">pip install langchain</CodeBlock>
  </TabItem>
  <TabItem value="conda" label="Conda">
    <CodeBlock language="bash">conda install langchain -c conda-forge</CodeBlock>
  </TabItem>
</Tabs>



For more details, see our [Installation guide](/docs/how_to/installation).

### LangSmith

Many of the applications you build with LangChain will contain multiple steps with multiple invocations of LLM calls.
As these applications get more and more complex, it becomes crucial to be able to inspect what exactly is going on inside your chain or agent.
The best way to do this is with [LangSmith](https://smith.langchain.com).

After you sign up at the link above, make sure to set your environment variables to start logging traces:

```shell
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="..."
```

Or, if in a notebook, you can set them with:

```python
import getpass
import os

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## Quickstart

First up, let's learn how to use a language model by itself. LangChain supports many different language models that you can use interchangably - select the one you want to use below!

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs openaiParams={`model="gpt-3.5-turbo"`} />

Let's first use the model directly. `ChatModel`s are instances of LangChain "Runnables", which means they expose a standard interface for interacting with them. To just simply call the model, we can pass in a list of messages to the `.invoke` method.


```python
from langchain_core.messages import HumanMessage

model.invoke([HumanMessage(content="Hi! I'm Bob")])
```



```output
AIMessage(content='Hello Bob! How can I assist you today?', response_metadata={'token_usage': {'completion_tokens': 10, 'prompt_tokens': 12, 'total_tokens': 22}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-d939617f-0c3b-45e9-a93f-13dafecbd4b5-0', usage_metadata={'input_tokens': 12, 'output_tokens': 10, 'total_tokens': 22})
```


The model on its own does not have any concept of state. For example, if you ask a followup question:


```python
model.invoke([HumanMessage(content="What's my name?")])
```



```output
AIMessage(content="I'm sorry, I don't have access to personal information unless you provide it to me. How may I assist you today?", response_metadata={'token_usage': {'completion_tokens': 26, 'prompt_tokens': 12, 'total_tokens': 38}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-47bc8c20-af7b-4fd2-9345-f0e9fdf18ce3-0', usage_metadata={'input_tokens': 12, 'output_tokens': 26, 'total_tokens': 38})
```


Let's take a look at the example [LangSmith trace](https://smith.langchain.com/public/5c21cb92-2814-4119-bae9-d02b8db577ac/r)

We can see that it doesn't take the previous conversation turn into context, and cannot answer the question.
This makes for a terrible chatbot experience!

To get around this, we need to pass the entire conversation history into the model. Let's see what happens when we do that:


```python
from langchain_core.messages import AIMessage

model.invoke(
    [
        HumanMessage(content="Hi! I'm Bob"),
        AIMessage(content="Hello Bob! How can I assist you today?"),
        HumanMessage(content="What's my name?"),
    ]
)
```



```output
AIMessage(content='Your name is Bob. How can I help you, Bob?', response_metadata={'token_usage': {'completion_tokens': 13, 'prompt_tokens': 35, 'total_tokens': 48}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-9f90291b-4df9-41dc-9ecf-1ee1081f4490-0', usage_metadata={'input_tokens': 35, 'output_tokens': 13, 'total_tokens': 48})
```


And now we can see that we get a good response!

This is the basic idea underpinning a chatbot's ability to interact conversationally.
So how do we best implement this?

## Message History

We can use a Message History class to wrap our model and make it stateful.
This will keep track of inputs and outputs of the model, and store them in some datastore.
Future interactions will then load those messages and pass them into the chain as part of the input.
Let's see how to use this!

First, let's make sure to install `langchain-community`, as we will be using an integration in there to store message history.


```python
# ! pip install langchain_community
```

After that, we can import the relevant classes and set up our chain which wraps the model and adds in this message history. A key part here is the function we pass into as the `get_session_history`. This function is expected to take in a `session_id` and return a Message History object. This `session_id` is used to distinguish between separate conversations, and should be passed in as part of the config when calling the new chain (we'll show how to do that).


```python
from langchain_core.chat_history import (
    BaseChatMessageHistory,
    InMemoryChatMessageHistory,
)
from langchain_core.runnables.history import RunnableWithMessageHistory

store = {}


def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]


with_message_history = RunnableWithMessageHistory(model, get_session_history)
```

We now need to create a `config` that we pass into the runnable every time. This config contains information that is not part of the input directly, but is still useful. In this case, we want to include a `session_id`. This should look like:


```python
config = {"configurable": {"session_id": "abc2"}}
```


```python
response = with_message_history.invoke(
    [HumanMessage(content="Hi! I'm Bob")],
    config=config,
)

response.content
```



```output
'Hi Bob! How can I assist you today?'
```



```python
response = with_message_history.invoke(
    [HumanMessage(content="What's my name?")],
    config=config,
)

response.content
```



```output
'Your name is Bob. How can I help you today, Bob?'
```


Great! Our chatbot now remembers things about us. If we change the config to reference a different `session_id`, we can see that it starts the conversation fresh.


```python
config = {"configurable": {"session_id": "abc3"}}

response = with_message_history.invoke(
    [HumanMessage(content="What's my name?")],
    config=config,
)

response.content
```



```output
"I'm sorry, I cannot determine your name as I am an AI assistant and do not have access to that information."
```


However, we can always go back to the original conversation (since we are persisting it in a database)


```python
config = {"configurable": {"session_id": "abc2"}}

response = with_message_history.invoke(
    [HumanMessage(content="What's my name?")],
    config=config,
)

response.content
```



```output
'Your name is Bob. How can I assist you today, Bob?'
```


This is how we can support a chatbot having conversations with many users!

Right now, all we've done is add a simple persistence layer around the model. We can start to make the more complicated and personalized by adding in a prompt template.

## Prompt templates

Prompt Templates help to turn raw user information into a format that the LLM can work with. In this case, the raw user input is just a message, which we are passing to the LLM. Let's now make that a bit more complicated. First, let's add in a system message with some custom instructions (but still taking messages as input). Next, we'll add in more input besides just the messages.

First, let's add in a system message. To do this, we will create a ChatPromptTemplate. We will utilize `MessagesPlaceholder` to pass all the messages in.


```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant. Answer all questions to the best of your ability.",
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)

chain = prompt | model
```

Note that this slightly changes the input type - rather than pass in a list of messages, we are now passing in a dictionary with a `messages` key where that contains a list of messages.


```python
response = chain.invoke({"messages": [HumanMessage(content="hi! I'm bob")]})

response.content
```



```output
'Hello Bob! How can I assist you today?'
```


We can now wrap this in the same Messages History object as before


```python
with_message_history = RunnableWithMessageHistory(chain, get_session_history)
```


```python
config = {"configurable": {"session_id": "abc5"}}
```


```python
response = with_message_history.invoke(
    [HumanMessage(content="Hi! I'm Jim")],
    config=config,
)

response.content
```



```output
'Hello, Jim! How can I assist you today?'
```



```python
response = with_message_history.invoke(
    [HumanMessage(content="What's my name?")],
    config=config,
)

response.content
```



```output
'Your name is Jim.'
```


Awesome! Let's now make our prompt a little bit more complicated. Let's assume that the prompt template now looks something like this:


```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant. Answer all questions to the best of your ability in {language}.",
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)

chain = prompt | model
```

Note that we have added a new `language` input to the prompt. We can now invoke the chain and pass in a language of our choice.


```python
response = chain.invoke(
    {"messages": [HumanMessage(content="hi! I'm bob")], "language": "Spanish"}
)

response.content
```



```output
'¡Hola, Bob! ¿En qué puedo ayudarte hoy?'
```


Let's now wrap this more complicated chain in a Message History class. This time, because there are multiple keys in the input, we need to specify the correct key to use to save the chat history.


```python
with_message_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="messages",
)
```


```python
config = {"configurable": {"session_id": "abc11"}}
```


```python
response = with_message_history.invoke(
    {"messages": [HumanMessage(content="hi! I'm todd")], "language": "Spanish"},
    config=config,
)

response.content
```



```output
'¡Hola Todd! ¿En qué puedo ayudarte hoy?'
```



```python
response = with_message_history.invoke(
    {"messages": [HumanMessage(content="whats my name?")], "language": "Spanish"},
    config=config,
)

response.content
```



```output
'Tu nombre es Todd.'
```


To help you understand what's happening internally, check out [this LangSmith trace](https://smith.langchain.com/public/f48fabb6-6502-43ec-8242-afc352b769ed/r)

## Managing Conversation History

One important concept to understand when building chatbots is how to manage conversation history. If left unmanaged, the list of messages will grow unbounded and potentially overflow the context window of the LLM. Therefore, it is important to add a step that limits the size of the messages you are passing in.

**Importantly, you will want to do this BEFORE the prompt template but AFTER you load previous messages from Message History.**

We can do this by adding a simple step in front of the prompt that modifies the `messages` key appropriately, and then wrap that new chain in the Message History class. 

LangChain comes with a few built-in helpers for [managing a list of messages](/docs/how_to/#messages). In this case we'll use the [trim_messages](/docs/how_to/trim_messages/) helper to reduce how many messages we're sending to the model. The trimmer allows us to specify how many tokens we want to keep, along with other parameters like if we want to always keep the system message and whether to allow partial messages:


```python
from langchain_core.messages import SystemMessage, trim_messages

trimmer = trim_messages(
    max_tokens=65,
    strategy="last",
    token_counter=model,
    include_system=True,
    allow_partial=False,
    start_on="human",
)

messages = [
    SystemMessage(content="you're a good assistant"),
    HumanMessage(content="hi! I'm bob"),
    AIMessage(content="hi!"),
    HumanMessage(content="I like vanilla ice cream"),
    AIMessage(content="nice"),
    HumanMessage(content="whats 2 + 2"),
    AIMessage(content="4"),
    HumanMessage(content="thanks"),
    AIMessage(content="no problem!"),
    HumanMessage(content="having fun?"),
    AIMessage(content="yes!"),
]

trimmer.invoke(messages)
```



```output
[SystemMessage(content="you're a good assistant"),
 HumanMessage(content='whats 2 + 2'),
 AIMessage(content='4'),
 HumanMessage(content='thanks'),
 AIMessage(content='no problem!'),
 HumanMessage(content='having fun?'),
 AIMessage(content='yes!')]
```


To  use it in our chain, we just need to run the trimmer before we pass the `messages` input to our prompt. 

Now if we try asking the model our name, it won't know it since we trimmed that part of the chat history:


```python
from operator import itemgetter

from langchain_core.runnables import RunnablePassthrough

chain = (
    RunnablePassthrough.assign(messages=itemgetter("messages") | trimmer)
    | prompt
    | model
)

response = chain.invoke(
    {
        "messages": messages + [HumanMessage(content="what's my name?")],
        "language": "English",
    }
)
response.content
```



```output
"I'm sorry, but I don't have access to your personal information. How can I assist you today?"
```


But if we ask about information that is within the last few messages, it remembers:


```python
response = chain.invoke(
    {
        "messages": messages + [HumanMessage(content="what math problem did i ask")],
        "language": "English",
    }
)
response.content
```



```output
'You asked "what\'s 2 + 2?"'
```


Let's now wrap this in the Message History


```python
with_message_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="messages",
)

config = {"configurable": {"session_id": "abc20"}}
```


```python
response = with_message_history.invoke(
    {
        "messages": messages + [HumanMessage(content="whats my name?")],
        "language": "English",
    },
    config=config,
)

response.content
```



```output
"I'm sorry, I don't have access to that information. How can I assist you today?"
```


As expected, the first message where we stated our name has been trimmed. Plus there's now two new messages in the chat history (our latest question and the latest response). This means that even more information that used to be accessible in our conversation history is no longer available! In this case our initial math question has been trimmed from the history as well, so the model no longer knows about it:


```python
response = with_message_history.invoke(
    {
        "messages": [HumanMessage(content="what math problem did i ask?")],
        "language": "English",
    },
    config=config,
)

response.content
```



```output
"You haven't asked a math problem yet. Feel free to ask any math-related question you have, and I'll be happy to help you with it."
```


If you take a look at LangSmith, you can see exactly what is happening under the hood in the [LangSmith trace](https://smith.langchain.com/public/a64b8b7c-1fd6-4dbb-b11a-47cd09a5e4f1/r).

## Streaming

Now we've got a function chatbot. However, one *really* important UX consideration for chatbot application is streaming. LLMs can sometimes take a while to respond, and so in order to improve the user experience one thing that most application do is stream back each token as it is generated. This allows the user to see progress.

It's actually super easy to do this!

All chains expose a `.stream` method, and ones that use message history are no different. We can simply use that method to get back a streaming response.


```python
config = {"configurable": {"session_id": "abc15"}}
for r in with_message_history.stream(
    {
        "messages": [HumanMessage(content="hi! I'm todd. tell me a joke")],
        "language": "English",
    },
    config=config,
):
    print(r.content, end="|")
```
```output
|Hi| Todd|!| Sure|,| here|'s| a| joke| for| you|:| Why| couldn|'t| the| bicycle| find| its| way| home|?| Because| it| lost| its| bearings|!| 😄||
```
## Next Steps

Now that you understand the basics of how to create a chatbot in LangChain, some more advanced tutorials you may be interested in are:

- [Conversational RAG](/docs/tutorials/qa_chat_history): Enable a chatbot experience over an external source of data
- [Agents](/docs/tutorials/agents): Build a chatbot that can take actions

If you want to dive deeper on specifics, some things worth checking out are:

- [Streaming](/docs/how_to/streaming): streaming is *crucial* for chat applications
- [How to add message history](/docs/how_to/message_history): for a deeper dive into all things related to message history
- [How to manage large message history](/docs/how_to/trim_messages/): more techniques for managing a large chat history
