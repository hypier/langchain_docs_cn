---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/redis_chat_message_history.ipynb
---

# Redis

>[Redis (远程字典服务器)](https://en.wikipedia.org/wiki/Redis) 是一个开源的内存存储，作为分布式内存键值数据库、缓存和消息代理使用，并具有可选的持久性。由于它将所有数据保存在内存中，并且由于其设计，`Redis` 提供了低延迟的读写操作，使其特别适合需要缓存的用例。Redis 是最受欢迎的 NoSQL 数据库，也是最受欢迎的数据库之一。

此笔记本介绍了如何使用 `Redis` 存储聊天消息历史记录。

## 设置
首先，我们需要安装依赖项，并使用类似于 `redis-server` 的命令启动一个 redis 实例。


```python
pip install -U langchain-community redis
```


```python
from langchain_community.chat_message_histories import RedisChatMessageHistory
```

## 存储和检索消息


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

## 在链中使用


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
        ("system", "您是一个助手。"),
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

chain_with_history.invoke({"question": "嗨！我是鲍勃"}, config=config)

chain_with_history.invoke({"question": "我叫什么名字"}, config=config)
```



```output
AIMessage(content='您的名字是鲍勃，正如您之前提到的。您还有什么具体需要帮助的吗，鲍勃？')
```