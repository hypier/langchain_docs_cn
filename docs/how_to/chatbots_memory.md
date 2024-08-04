---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/chatbots_memory.ipynb
sidebar_position: 1
---
# How to add memory to chatbots

A key feature of chatbots is their ability to use content of previous conversation turns as context. This state management can take several forms, including:

- Simply stuffing previous messages into a chat model prompt.
- The above, but trimming old messages to reduce the amount of distracting information the model has to deal with.
- More complex modifications like synthesizing summaries for long running conversations.

We'll go into more detail on a few techniques below!

## Setup

You'll need to install a few packages, and have your OpenAI API key set as an environment variable named `OPENAI_API_KEY`:


```python
%pip install --upgrade --quiet langchain langchain-openai

# Set env var OPENAI_API_KEY or load from a .env file:
import dotenv

dotenv.load_dotenv()
```
```output
[33mWARNING: You are using pip version 22.0.4; however, version 23.3.2 is available.
You should consider upgrading via the '/Users/jacoblee/.pyenv/versions/3.10.5/bin/python -m pip install --upgrade pip' command.[0m[33m
[0mNote: you may need to restart the kernel to use updated packages.
```


```output
True
```


Let's also set up a chat model that we'll use for the below examples.


```python
from langchain_openai import ChatOpenAI

chat = ChatOpenAI(model="gpt-3.5-turbo-0125")
```

## Message passing

The simplest form of memory is simply passing chat history messages into a chain. Here's an example:


```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant. Answer all questions to the best of your ability.",
        ),
        ("placeholder", "{messages}"),
    ]
)

chain = prompt | chat

ai_msg = chain.invoke(
    {
        "messages": [
            (
                "human",
                "Translate this sentence from English to French: I love programming.",
            ),
            ("ai", "J'adore la programmation."),
            ("human", "What did you just say?"),
        ],
    }
)
print(ai_msg.content)
```
```output
I said "J'adore la programmation," which means "I love programming" in French.
```
We can see that by passing the previous conversation into a chain, it can use it as context to answer questions. This is the basic concept underpinning chatbot memory - the rest of the guide will demonstrate convenient techniques for passing or reformatting messages.

## Chat history

It's perfectly fine to store and pass messages directly as an array, but we can use LangChain's built-in [message history class](https://api.python.langchain.com/en/latest/langchain_api_reference.html#module-langchain.memory) to store and load messages as well. Instances of this class are responsible for storing and loading chat messages from persistent storage. LangChain integrates with many providers - you can see a [list of integrations here](/docs/integrations/memory) - but for this demo we will use an ephemeral demo class.

Here's an example of the API:


```python
from langchain_community.chat_message_histories import ChatMessageHistory

demo_ephemeral_chat_history = ChatMessageHistory()

demo_ephemeral_chat_history.add_user_message(
    "Translate this sentence from English to French: I love programming."
)

demo_ephemeral_chat_history.add_ai_message("J'adore la programmation.")

demo_ephemeral_chat_history.messages
```



```output
[HumanMessage(content='Translate this sentence from English to French: I love programming.'),
 AIMessage(content="J'adore la programmation.")]
```


We can use it directly to store conversation turns for our chain:


```python
demo_ephemeral_chat_history = ChatMessageHistory()

input1 = "Translate this sentence from English to French: I love programming."

demo_ephemeral_chat_history.add_user_message(input1)

response = chain.invoke(
    {
        "messages": demo_ephemeral_chat_history.messages,
    }
)

demo_ephemeral_chat_history.add_ai_message(response)

input2 = "What did I just ask you?"

demo_ephemeral_chat_history.add_user_message(input2)

chain.invoke(
    {
        "messages": demo_ephemeral_chat_history.messages,
    }
)
```



```output
AIMessage(content='You just asked me to translate the sentence "I love programming" from English to French.', response_metadata={'token_usage': {'completion_tokens': 18, 'prompt_tokens': 61, 'total_tokens': 79}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-5cbb21c2-9c30-4031-8ea8-bfc497989535-0', usage_metadata={'input_tokens': 61, 'output_tokens': 18, 'total_tokens': 79})
```


## Automatic history management

The previous examples pass messages to the chain explicitly. This is a completely acceptable approach, but it does require external management of new messages. LangChain also includes an wrapper for LCEL chains that can handle this process automatically called `RunnableWithMessageHistory`.

To show how it works, let's slightly modify the above prompt to take a final `input` variable that populates a `HumanMessage` template after the chat history. This means that we will expect a `chat_history` parameter that contains all messages BEFORE the current messages instead of all messages:


```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant. Answer all questions to the best of your ability.",
        ),
        ("placeholder", "{chat_history}"),
        ("human", "{input}"),
    ]
)

chain = prompt | chat
```

 We'll pass the latest input to the conversation here and let the `RunnableWithMessageHistory` class wrap our chain and do the work of appending that `input` variable to the chat history.
 
 Next, let's declare our wrapped chain:


```python
from langchain_core.runnables.history import RunnableWithMessageHistory

demo_ephemeral_chat_history_for_chain = ChatMessageHistory()

chain_with_message_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: demo_ephemeral_chat_history_for_chain,
    input_messages_key="input",
    history_messages_key="chat_history",
)
```

This class takes a few parameters in addition to the chain that we want to wrap:

- A factory function that returns a message history for a given session id. This allows your chain to handle multiple users at once by loading different messages for different conversations.
- An `input_messages_key` that specifies which part of the input should be tracked and stored in the chat history. In this example, we want to track the string passed in as `input`.
- A `history_messages_key` that specifies what the previous messages should be injected into the prompt as. Our prompt has a `MessagesPlaceholder` named `chat_history`, so we specify this property to match.
- (For chains with multiple outputs) an `output_messages_key` which specifies which output to store as history. This is the inverse of `input_messages_key`.

We can invoke this new chain as normal, with an additional `configurable` field that specifies the particular `session_id` to pass to the factory function. This is unused for the demo, but in real-world chains, you'll want to return a chat history corresponding to the passed session:


```python
chain_with_message_history.invoke(
    {"input": "Translate this sentence from English to French: I love programming."},
    {"configurable": {"session_id": "unused"}},
)
```
```output
Parent run dc4e2f79-4bcd-4a36-9506-55ace9040588 not found for run 34b5773e-3ced-46a6-8daf-4d464c15c940. Treating as a root run.
```


```output
AIMessage(content='"J\'adore la programmation."', response_metadata={'token_usage': {'completion_tokens': 9, 'prompt_tokens': 39, 'total_tokens': 48}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-648b0822-b0bb-47a2-8e7d-7d34744be8f2-0', usage_metadata={'input_tokens': 39, 'output_tokens': 9, 'total_tokens': 48})
```



```python
chain_with_message_history.invoke(
    {"input": "What did I just ask you?"}, {"configurable": {"session_id": "unused"}}
)
```
```output
Parent run cc14b9d8-c59e-40db-a523-d6ab3fc2fa4f not found for run 5b75e25c-131e-46ee-9982-68569db04330. Treating as a root run.
```


```output
AIMessage(content='You asked me to translate the sentence "I love programming" from English to French.', response_metadata={'token_usage': {'completion_tokens': 17, 'prompt_tokens': 63, 'total_tokens': 80}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-5950435c-1dc2-43a6-836f-f989fd62c95e-0', usage_metadata={'input_tokens': 63, 'output_tokens': 17, 'total_tokens': 80})
```


## Modifying chat history

Modifying stored chat messages can help your chatbot handle a variety of situations. Here are some examples:

### Trimming messages

LLMs and chat models have limited context windows, and even if you're not directly hitting limits, you may want to limit the amount of distraction the model has to deal with. One solution is trim the historic messages before passing them to the model. Let's use an example history with some preloaded messages:


```python
demo_ephemeral_chat_history = ChatMessageHistory()

demo_ephemeral_chat_history.add_user_message("Hey there! I'm Nemo.")
demo_ephemeral_chat_history.add_ai_message("Hello!")
demo_ephemeral_chat_history.add_user_message("How are you today?")
demo_ephemeral_chat_history.add_ai_message("Fine thanks!")

demo_ephemeral_chat_history.messages
```



```output
[HumanMessage(content="Hey there! I'm Nemo."),
 AIMessage(content='Hello!'),
 HumanMessage(content='How are you today?'),
 AIMessage(content='Fine thanks!')]
```


Let's use this message history with the `RunnableWithMessageHistory` chain we declared above:


```python
chain_with_message_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: demo_ephemeral_chat_history,
    input_messages_key="input",
    history_messages_key="chat_history",
)

chain_with_message_history.invoke(
    {"input": "What's my name?"},
    {"configurable": {"session_id": "unused"}},
)
```
```output
Parent run 7ff2d8ec-65e2-4f67-8961-e498e2c4a591 not found for run 3881e990-6596-4326-84f6-2b76949e0657. Treating as a root run.
```


```output
AIMessage(content='Your name is Nemo.', response_metadata={'token_usage': {'completion_tokens': 6, 'prompt_tokens': 66, 'total_tokens': 72}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-f8aabef8-631a-4238-a39b-701e881fbe47-0', usage_metadata={'input_tokens': 66, 'output_tokens': 6, 'total_tokens': 72})
```


We can see the chain remembers the preloaded name.

But let's say we have a very small context window, and we want to trim the number of messages passed to the chain to only the 2 most recent ones. We can use the built in [trim_messages](/docs/how_to/trim_messages/) util to trim messages based on their token count before they reach our prompt. In this case we'll count each message as 1 "token" and keep only the last two messages:


```python
from operator import itemgetter

from langchain_core.messages import trim_messages
from langchain_core.runnables import RunnablePassthrough

trimmer = trim_messages(strategy="last", max_tokens=2, token_counter=len)

chain_with_trimming = (
    RunnablePassthrough.assign(chat_history=itemgetter("chat_history") | trimmer)
    | prompt
    | chat
)

chain_with_trimmed_history = RunnableWithMessageHistory(
    chain_with_trimming,
    lambda session_id: demo_ephemeral_chat_history,
    input_messages_key="input",
    history_messages_key="chat_history",
)
```

Let's call this new chain and check the messages afterwards:


```python
chain_with_trimmed_history.invoke(
    {"input": "Where does P. Sherman live?"},
    {"configurable": {"session_id": "unused"}},
)
```
```output
Parent run 775cde65-8d22-4c44-80bb-f0b9811c32ca not found for run 5cf71d0e-4663-41cd-8dbe-e9752689cfac. Treating as a root run.
```


```output
AIMessage(content='P. Sherman is a fictional character from the animated movie "Finding Nemo" who lives at 42 Wallaby Way, Sydney.', response_metadata={'token_usage': {'completion_tokens': 27, 'prompt_tokens': 53, 'total_tokens': 80}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-5642ef3a-fdbe-43cf-a575-d1785976a1b9-0', usage_metadata={'input_tokens': 53, 'output_tokens': 27, 'total_tokens': 80})
```



```python
demo_ephemeral_chat_history.messages
```



```output
[HumanMessage(content="Hey there! I'm Nemo."),
 AIMessage(content='Hello!'),
 HumanMessage(content='How are you today?'),
 AIMessage(content='Fine thanks!'),
 HumanMessage(content="What's my name?"),
 AIMessage(content='Your name is Nemo.', response_metadata={'token_usage': {'completion_tokens': 6, 'prompt_tokens': 66, 'total_tokens': 72}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-f8aabef8-631a-4238-a39b-701e881fbe47-0', usage_metadata={'input_tokens': 66, 'output_tokens': 6, 'total_tokens': 72}),
 HumanMessage(content='Where does P. Sherman live?'),
 AIMessage(content='P. Sherman is a fictional character from the animated movie "Finding Nemo" who lives at 42 Wallaby Way, Sydney.', response_metadata={'token_usage': {'completion_tokens': 27, 'prompt_tokens': 53, 'total_tokens': 80}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-5642ef3a-fdbe-43cf-a575-d1785976a1b9-0', usage_metadata={'input_tokens': 53, 'output_tokens': 27, 'total_tokens': 80})]
```


And we can see that our history has removed the two oldest messages while still adding the most recent conversation at the end. The next time the chain is called, `trim_messages` will be called again, and only the two most recent messages will be passed to the model. In this case, this means that the model will forget the name we gave it the next time we invoke it:


```python
chain_with_trimmed_history.invoke(
    {"input": "What is my name?"},
    {"configurable": {"session_id": "unused"}},
)
```
```output
Parent run fde7123f-6fd3-421a-a3fc-2fb37dead119 not found for run 061a4563-2394-470d-a3ed-9bf1388ca431. Treating as a root run.
```


```output
AIMessage(content="I'm sorry, but I don't have access to your personal information, so I don't know your name. How else may I assist you today?", response_metadata={'token_usage': {'completion_tokens': 31, 'prompt_tokens': 74, 'total_tokens': 105}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-0ab03495-1f7c-4151-9070-56d2d1c565ff-0', usage_metadata={'input_tokens': 74, 'output_tokens': 31, 'total_tokens': 105})
```


Check out our [how to guide on trimming messages](/docs/how_to/trim_messages/) for more.

### Summary memory

We can use this same pattern in other ways too. For example, we could use an additional LLM call to generate a summary of the conversation before calling our chain. Let's recreate our chat history and chatbot chain:


```python
demo_ephemeral_chat_history = ChatMessageHistory()

demo_ephemeral_chat_history.add_user_message("Hey there! I'm Nemo.")
demo_ephemeral_chat_history.add_ai_message("Hello!")
demo_ephemeral_chat_history.add_user_message("How are you today?")
demo_ephemeral_chat_history.add_ai_message("Fine thanks!")

demo_ephemeral_chat_history.messages
```



```output
[HumanMessage(content="Hey there! I'm Nemo."),
 AIMessage(content='Hello!'),
 HumanMessage(content='How are you today?'),
 AIMessage(content='Fine thanks!')]
```


We'll slightly modify the prompt to make the LLM aware that will receive a condensed summary instead of a chat history:


```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant. Answer all questions to the best of your ability. The provided chat history includes facts about the user you are speaking with.",
        ),
        ("placeholder", "{chat_history}"),
        ("user", "{input}"),
    ]
)

chain = prompt | chat

chain_with_message_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: demo_ephemeral_chat_history,
    input_messages_key="input",
    history_messages_key="chat_history",
)
```

And now, let's create a function that will distill previous interactions into a summary. We can add this one to the front of the chain too:


```python
def summarize_messages(chain_input):
    stored_messages = demo_ephemeral_chat_history.messages
    if len(stored_messages) == 0:
        return False
    summarization_prompt = ChatPromptTemplate.from_messages(
        [
            ("placeholder", "{chat_history}"),
            (
                "user",
                "Distill the above chat messages into a single summary message. Include as many specific details as you can.",
            ),
        ]
    )
    summarization_chain = summarization_prompt | chat

    summary_message = summarization_chain.invoke({"chat_history": stored_messages})

    demo_ephemeral_chat_history.clear()

    demo_ephemeral_chat_history.add_message(summary_message)

    return True


chain_with_summarization = (
    RunnablePassthrough.assign(messages_summarized=summarize_messages)
    | chain_with_message_history
)
```

Let's see if it remembers the name we gave it:


```python
chain_with_summarization.invoke(
    {"input": "What did I say my name was?"},
    {"configurable": {"session_id": "unused"}},
)
```



```output
AIMessage(content='You introduced yourself as Nemo. How can I assist you today, Nemo?')
```



```python
demo_ephemeral_chat_history.messages
```



```output
[AIMessage(content='The conversation is between Nemo and an AI. Nemo introduces himself and the AI responds with a greeting. Nemo then asks the AI how it is doing, and the AI responds that it is fine.'),
 HumanMessage(content='What did I say my name was?'),
 AIMessage(content='You introduced yourself as Nemo. How can I assist you today, Nemo?')]
```


Note that invoking the chain again will generate another summary generated from the initial summary plus new messages and so on. You could also design a hybrid approach where a certain number of messages are retained in chat history while others are summarized.
