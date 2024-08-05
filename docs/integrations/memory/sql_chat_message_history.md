---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/sql_chat_message_history.ipynb
---

# SQL (SQLAlchemy)

>[结构化查询语言 (SQL)](https://en.wikipedia.org/wiki/SQL) 是一种特定领域语言，用于编程，旨在管理关系数据库管理系统 (RDBMS) 中的数据，或在关系数据流管理系统 (RDSMS) 中进行流处理。它在处理结构化数据方面特别有用，即包含实体和变量之间关系的数据。

>[SQLAlchemy](https://github.com/sqlalchemy/sqlalchemy) 是一个开源的 `SQL` 工具包和对象关系映射器 (ORM)，用于 Python 编程语言，采用 MIT 许可证发布。

本笔记本介绍了一个 `SQLChatMessageHistory` 类，允许在任何由 `SQLAlchemy` 支持的数据库中存储聊天记录。

请注意，要在除 `SQLite` 以外的数据库中使用它，您需要安装相应的数据库驱动程序。

## 设置

集成存在于 `langchain-community` 包中，因此我们需要安装它。我们还需要安装 `SQLAlchemy` 包。

```bash
pip install -U langchain-community SQLAlchemy langchain-openai
```

设置 [LangSmith](https://smith.langchain.com/) 以实现最佳的可观察性也是有帮助的（但不是必需的）。

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 使用方法

要使用存储，您只需提供两样东西：

1. 会话 ID - 会话的唯一标识符，如用户名、电子邮件、聊天 ID 等。
2. 连接字符串 - 指定数据库连接的字符串。它将传递给 SQLAlchemy 的 create_engine 函数。


```python
from langchain_community.chat_message_histories import SQLChatMessageHistory

chat_message_history = SQLChatMessageHistory(
    session_id="test_session", connection_string="sqlite:///sqlite.db"
)

chat_message_history.add_user_message("Hello")
chat_message_history.add_ai_message("Hi")
```


```python
chat_message_history.messages
```



```output
[HumanMessage(content='Hello'), AIMessage(content='Hi')]
```

## 链接

我们可以轻松地将此消息历史类与 [LCEL Runnables](/docs/how_to/message_history) 结合起来。

为此，我们需要使用 OpenAI，因此需要安装它。

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI
```

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)

chain = prompt | ChatOpenAI()
```

```python
chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: SQLChatMessageHistory(
        session_id=session_id, connection_string="sqlite:///sqlite.db"
    ),
    input_messages_key="question",
    history_messages_key="history",
)
```

```python
# 这是我们配置会话 ID 的地方
config = {"configurable": {"session_id": "<SESSION_ID>"}}
```

```python
chain_with_history.invoke({"question": "Hi! I'm bob"}, config=config)
```

```output
AIMessage(content='Hello Bob! How can I assist you today?')
```

```python
chain_with_history.invoke({"question": "Whats my name"}, config=config)
```

```output
AIMessage(content='Your name is Bob! Is there anything specific you would like assistance with, Bob?')
```