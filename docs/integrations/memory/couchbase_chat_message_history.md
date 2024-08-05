---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/couchbase_chat_message_history.ipynb
---

# Couchbase
> Couchbase 是一个屡获殊荣的分布式 NoSQL 云数据库，提供无与伦比的多功能性、性能、可扩展性和财务价值，适用于您的所有云、移动、人工智能和边缘计算应用。Couchbase 通过为开发者提供编码辅助和为其应用提供向量搜索来拥抱人工智能。

本笔记本介绍如何使用 `CouchbaseChatMessageHistory` 类在 Couchbase 集群中存储聊天消息历史。

## 设置 Couchbase 集群
要运行此演示，您需要一个 Couchbase 集群。

您可以使用 [Couchbase Capella](https://www.couchbase.com/products/capella/) 和您自我管理的 Couchbase Server。

## 安装依赖
`CouchbaseChatMessageHistory` 位于 `langchain-couchbase` 包中。 


```python
%pip install --upgrade --quiet langchain-couchbase
```
```output
注意：您可能需要重启内核以使用更新的包。
```

## 创建 Couchbase 连接对象
我们首先创建一个与 Couchbase 集群的连接，然后将集群对象传递给向量存储。

在这里，我们使用用户名和密码进行连接。您也可以通过其他任何支持的方式连接到您的集群。

有关连接到 Couchbase 集群的更多信息，请查看 [Python SDK 文档](https://docs.couchbase.com/python-sdk/current/hello-world/start-using-sdk.html#connect)。

```python
COUCHBASE_CONNECTION_STRING = (
    "couchbase://localhost"  # or "couchbases://localhost" if using TLS
)
DB_USERNAME = "Administrator"
DB_PASSWORD = "Password"
```

```python
from datetime import timedelta

from couchbase.auth import PasswordAuthenticator
from couchbase.cluster import Cluster
from couchbase.options import ClusterOptions

auth = PasswordAuthenticator(DB_USERNAME, DB_PASSWORD)
options = ClusterOptions(auth)
cluster = Cluster(COUCHBASE_CONNECTION_STRING, options)

# Wait until the cluster is ready for use.
cluster.wait_until_ready(timedelta(seconds=5))
```

我们现在将在 Couchbase 集群中设置要用于存储消息历史记录的桶、范围和集合名称。

请注意，桶、范围和集合需要在使用它们存储消息历史记录之前存在。

```python
BUCKET_NAME = "langchain-testing"
SCOPE_NAME = "_default"
COLLECTION_NAME = "conversational_cache"
```

## 使用方法
为了存储消息，您需要以下内容：
- Couchbase Cluster object: 有效连接到Couchbase集群
- bucket_name: 存储聊天消息历史的集群中的桶
- scope_name: 存储消息历史的桶中的范围
- collection_name: 存储消息历史的范围中的集合
- session_id: 会话的唯一标识符

可选地，您可以配置以下内容：
- session_id_key: 用于存储`session_id`的聊天消息文档中的字段
- message_key: 用于存储消息内容的聊天消息文档中的字段
- create_index: 用于指定是否需要在集合上创建索引。默认情况下，会在文档的`message_key`和`session_id_key`上创建索引


```python
from langchain_couchbase.chat_message_histories import CouchbaseChatMessageHistory

message_history = CouchbaseChatMessageHistory(
    cluster=cluster,
    bucket_name=BUCKET_NAME,
    scope_name=SCOPE_NAME,
    collection_name=COLLECTION_NAME,
    session_id="test-session",
)

message_history.add_user_message("hi!")

message_history.add_ai_message("how are you doing?")
```


```python
message_history.messages
```



```output
[HumanMessage(content='hi!'), AIMessage(content='how are you doing?')]
```

## 链接
聊天消息历史类可以与 [LCEL Runnables](https://python.langchain.com/v0.2/docs/how_to/message_history/) 一起使用


```python
import getpass
import os

from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI

os.environ["OPENAI_API_KEY"] = getpass.getpass()
```


```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)

# Create the LCEL runnable
chain = prompt | ChatOpenAI()
```


```python
chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: CouchbaseChatMessageHistory(
        cluster=cluster,
        bucket_name=BUCKET_NAME,
        scope_name=SCOPE_NAME,
        collection_name=COLLECTION_NAME,
        session_id=session_id,
    ),
    input_messages_key="question",
    history_messages_key="history",
)
```


```python
# This is where we configure the session id
config = {"configurable": {"session_id": "testing"}}
```


```python
chain_with_history.invoke({"question": "Hi! I'm bob"}, config=config)
```



```output
AIMessage(content='Hello Bob! How can I assist you today?', response_metadata={'token_usage': {'completion_tokens': 10, 'prompt_tokens': 22, 'total_tokens': 32}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-a0f8a29e-ddf4-4e06-a1fe-cf8c325a2b72-0', usage_metadata={'input_tokens': 22, 'output_tokens': 10, 'total_tokens': 32})
```



```python
chain_with_history.invoke({"question": "Whats my name"}, config=config)
```



```output
AIMessage(content='Your name is Bob.', response_metadata={'token_usage': {'completion_tokens': 5, 'prompt_tokens': 43, 'total_tokens': 48}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-f764a9eb-999e-4042-96b6-fe47b7ae4779-0', usage_metadata={'input_tokens': 43, 'output_tokens': 5, 'total_tokens': 48})
```