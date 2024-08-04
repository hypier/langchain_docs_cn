---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/redis_chat_message_history.ipynb
---
# Redis

>[Redis (Remote Dictionary Server)](https://en.wikipedia.org/wiki/Redis) is an open-source in-memory storage, used as a distributed, in-memory key–value database, cache and message broker, with optional durability. Because it holds all data in memory and because of its design, `Redis` offers low-latency reads and writes, making it particularly suitable for use cases that require a cache. Redis is the most popular NoSQL database, and one of the most popular databases overall.

This notebook goes over how to use `Redis` to store chat message history.

## Setup
First we need to install dependencies, and start a redis instance using commands like: `redis-server`.


```python
pip install -U langchain-community redis
```


```python
from langchain_community.chat_message_histories import RedisChatMessageHistory
```

## Store and Retrieve Messages


```python
history = RedisChatMessageHistory("foo", url="redis://localhost:6379")

history.add_user_message("hi!")

history.add_ai_message("whats up?")
```


```python
history.messages
```



```output
[HumanMessage(content='hi!'), AIMessage(content='whats up?')]
```


## Using in the Chains


```python
pip install -U langchain-openai
```


```python
from typing import Optional

from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI
```


```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You're an assistant。"),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)

chain = prompt | ChatOpenAI()

chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: RedisChatMessageHistory(
        session_id, url="redis://localhost:6379"
    ),
    input_messages_key="question",
    history_messages_key="history",
)

config = {"configurable": {"session_id": "foo"}}

chain_with_history.invoke({"question": "Hi! I'm bob"}, config=config)

chain_with_history.invoke({"question": "Whats my name"}, config=config)
```



```output
AIMessage(content='Your name is Bob, as you mentioned earlier. Is there anything specific you would like assistance with, Bob?')
```

