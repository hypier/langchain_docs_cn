---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/tidb_chat_message_history.ipynb
---

# TiDB

> [TiDB Cloud](https://www.pingcap.com/tidb-serverless/)，是一种综合的数据库即服务（DBaaS）解决方案，提供专用和无服务器选项。TiDB Serverless 目前正在将内置向量搜索集成到 MySQL 生态系统中。通过这一增强功能，您可以无缝地使用 TiDB Serverless 开发 AI 应用，而无需新的数据库或额外的技术栈。创建一个免费的 TiDB Serverless 集群，并在 https://pingcap.com/ai 开始使用向量搜索功能。

本笔记本介绍如何使用 TiDB 存储聊天消息历史记录。

## 设置

首先，我们将安装以下依赖项：


```python
%pip install --upgrade --quiet langchain langchain_openai langchain-community
```

配置您的 OpenAI 密钥


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("Input your OpenAI API key:")
```

最后，我们将配置与 TiDB 的连接。在这个笔记本中，我们将遵循 TiDB Cloud 提供的标准连接方法，以建立安全高效的数据库连接。


```python
# copy from tidb cloud console
tidb_connection_string_template = "mysql+pymysql://<USER>:<PASSWORD>@<HOST>:4000/<DB>?ssl_ca=/etc/ssl/cert.pem&ssl_verify_cert=true&ssl_verify_identity=true"
tidb_password = getpass.getpass("Input your TiDB password:")
tidb_connection_string = tidb_connection_string_template.replace(
    "<PASSWORD>", tidb_password
)
```

## 生成历史数据

创建一组历史数据，作为我们即将进行的演示的基础。

```python
from datetime import datetime

from langchain_community.chat_message_histories import TiDBChatMessageHistory

history = TiDBChatMessageHistory(
    connection_string=tidb_connection_string,
    session_id="code_gen",
    earliest_time=datetime.utcnow(),  # Optional to set earliest_time to load messages after this time point.
)

history.add_user_message("How's our feature going?")
history.add_ai_message(
    "It's going well. We are working on testing now. It will be released in Feb."
)
```

```python
history.messages
```

```output
[HumanMessage(content="How's our feature going?"),
 AIMessage(content="It's going well. We are working on testing now. It will be released in Feb.")]
```

## 使用历史数据聊天

让我们基于之前生成的历史数据创建一个动态的聊天互动。

首先，使用 LangChain 创建聊天链：

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You're an assistant who's good at coding. You're helping a startup build",
        ),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)
chain = prompt | ChatOpenAI()
```

基于历史数据构建可运行对象：

```python
from langchain_core.runnables.history import RunnableWithMessageHistory

chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: TiDBChatMessageHistory(
        session_id=session_id, connection_string=tidb_connection_string
    ),
    input_messages_key="question",
    history_messages_key="history",
)
```

启动聊天：

```python
response = chain_with_history.invoke(
    {"question": "Today is Jan 1st. How many days until our feature is released?"},
    config={"configurable": {"session_id": "code_gen"}},
)
response
```

```output
AIMessage(content='There are 31 days in January, so there are 30 days until our feature is released in February.')
```

## 检查历史数据


```python
history.reload_cache()
history.messages
```



```output
[HumanMessage(content="How's our feature going?"),
 AIMessage(content="It's going well. We are working on testing now. It will be released in Feb."),
 HumanMessage(content='Today is Jan 1st. How many days until our feature is released?'),
 AIMessage(content='There are 31 days in January, so there are 30 days until our feature is released in February.')]
```