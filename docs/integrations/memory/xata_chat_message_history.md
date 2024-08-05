---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/xata_chat_message_history.ipynb
---

# Xata

>[Xata](https://xata.io) 是一个无服务器的数据平台，基于 `PostgreSQL` 和 `Elasticsearch`。它提供了一个用于与数据库交互的 Python SDK，以及一个用于管理数据的用户界面。通过 `XataChatMessageHistory` 类，您可以使用 Xata 数据库来长期保存聊天会话。

本笔记本涵盖：

* 一个简单的示例，展示 `XataChatMessageHistory` 的功能。
* 一个更复杂的示例，使用 REACT 代理，根据知识库或文档（以向量存储形式存储在 Xata 中）回答问题，并且还拥有其过去消息的长期可搜索历史（以内存存储形式存储在 Xata 中）。

## 设置

### 创建数据库

在 [Xata UI](https://app.xata.io) 中创建一个新的数据库。您可以随意命名，在这个记事本中我们将使用 `langchain`。Langchain 集成可以自动创建用于存储记忆的表，这就是我们在本示例中将使用的。如果您想预先创建表，请确保它具有正确的模式，并在创建类时将 `create_table` 设置为 `False`。预先创建表可以在每次会话初始化期间节省一次与数据库的往返。

首先，让我们安装我们的依赖项：

```python
%pip install --upgrade --quiet  xata langchain-openai langchain langchain-community
```

接下来，我们需要获取 Xata 的环境变量。您可以通过访问您的 [帐户设置](https://app.xata.io/settings) 创建一个新的 API 密钥。要查找数据库 URL，请转到您创建的数据库的设置页面。数据库 URL 应该类似于： `https://demo-uni3q8.eu-west-1.xata.sh/db/langchain`。

```python
import getpass

api_key = getpass.getpass("Xata API key: ")
db_url = input("Xata database URL (copy it from your DB settings):")
```

## 创建简单的内存存储

为了单独测试内存存储功能，我们使用以下代码片段：

```python
from langchain_community.chat_message_histories import XataChatMessageHistory

history = XataChatMessageHistory(
    session_id="session-1", api_key=api_key, db_url=db_url, table_name="memory"
)

history.add_user_message("hi!")

history.add_ai_message("whats up?")
```

上述代码创建了一个会话，ID 为 `session-1`，并在其中存储了两条消息。运行上述代码后，如果您访问 Xata UI，您应该会看到一个名为 `memory` 的表格，以及添加的两条消息。

您可以使用以下代码检索特定会话的消息历史记录：

```python
history.messages
```

## 带记忆的数据对话问答链

现在让我们看看一个更复杂的例子，在这个例子中，我们结合了OpenAI、Xata Vector Store集成和Xata内存存储集成，创建一个基于您数据的问答聊天机器人，支持后续问题和历史记录。

我们需要访问OpenAI API，因此让我们配置API密钥：

```python
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

为了存储聊天机器人将搜索答案的文档，请使用Xata UI向您的`langchain`数据库添加一个名为`docs`的表，并添加以下列：

* `content` 类型为 "Text"。用于存储 `Document.pageContent` 的值。
* `embedding` 类型为 "Vector"。使用您计划使用的模型所使用的维度。在本笔记本中，我们使用OpenAI嵌入，具有1536个维度。

让我们创建向量存储并添加一些示例文档：

```python
from langchain_community.vectorstores.xata import XataVectorStore
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()

texts = [
    "Xata is a Serverless Data platform based on PostgreSQL",
    "Xata offers a built-in vector type that can be used to store and query vectors",
    "Xata includes similarity search",
]

vector_store = XataVectorStore.from_texts(
    texts, embeddings, api_key=api_key, db_url=db_url, table_name="docs"
)
```

运行上述命令后，如果您访问Xata UI，您应该会看到文档及其嵌入加载在`docs`表中。

现在让我们创建一个ConversationBufferMemory来存储用户和AI的聊天消息。

```python
from uuid import uuid4

from langchain.memory import ConversationBufferMemory

chat_memory = XataChatMessageHistory(
    session_id=str(uuid4()),  # 每个用户会话需要唯一
    api_key=api_key,
    db_url=db_url,
    table_name="memory",
)
memory = ConversationBufferMemory(
    memory_key="chat_history", chat_memory=chat_memory, return_messages=True
)
```

现在是时候创建一个Agent，以便将向量存储和聊天记忆结合使用。

```python
from langchain.agents import AgentType, initialize_agent
from langchain.agents.agent_toolkits import create_retriever_tool
from langchain_openai import ChatOpenAI

tool = create_retriever_tool(
    vector_store.as_retriever(),
    "search_docs",
    "Searches and returns documents from the Xata manual. Useful when you need to answer questions about Xata.",
)
tools = [tool]

llm = ChatOpenAI(temperature=0)

agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.CHAT_CONVERSATIONAL_REACT_DESCRIPTION,
    verbose=True,
    memory=memory,
)
```

为了测试，让我们告诉代理我们的名字：

```python
agent.run(input="My name is bob")
```

现在，让我们问代理一些关于Xata的问题：

```python
agent.run(input="What is xata?")
```

请注意，它是根据存储在文档存储中的数据进行回答的。现在，让我们问一个后续问题：

```python
agent.run(input="Does it support similarity search?")
```

现在让我们测试它的记忆：

```python
agent.run(input="Did I tell you my name? What is it?")
```