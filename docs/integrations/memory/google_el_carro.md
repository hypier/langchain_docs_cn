---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/google_el_carro.ipynb
---

# Google El Carro Oracle

> [Google Cloud El Carro Oracle](https://github.com/GoogleCloudPlatform/elcarro-oracle-operator) 提供了一种在 `Kubernetes` 中运行 `Oracle` 数据库的方法，作为一个可移植的、开源的、社区驱动的、无供应商锁定的容器编排系统。`El Carro` 提供了强大的声明式 API，用于全面和一致的配置和部署，以及实时操作和监控。通过利用 `El Carro` Langchain 集成，扩展您的 `Oracle` 数据库的功能，以构建 AI 驱动的体验。

本指南介绍了如何使用 `El Carro` Langchain 集成来存储聊天消息历史，使用 `ElCarroChatMessageHistory` 类。此集成适用于任何 `Oracle` 数据库，无论其运行在哪里。

在 [GitHub](https://github.com/googleapis/langchain-google-el-carro-python/) 上了解更多关于该包的信息。

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-el-carro-python/blob/main/docs/chat_message_history.ipynb)

## 开始之前

要运行此笔记本，您需要执行以下操作：

 * 如果您希望使用 El Carro 运行 Oracle 数据库，请完成 [入门](https://github.com/googleapis/langchain-google-el-carro-python/tree/main/README.md#getting-started) 部分。

### 🦜🔗 库安装
集成在其自己的 `langchain-google-el-carro` 包中，因此我们需要安装它。

```python
%pip install --upgrade --quiet langchain-google-el-carro langchain-google-vertexai langchain
```

**仅限 Colab:** 取消注释以下单元以重启内核，或使用按钮重启内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重启终端。

```python
# # Automatically restart kernel after installs so that your environment can access the new packages
# import IPython

# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### 🔐 身份验证
以登录此笔记本的 IAM 用户身份对 Google Cloud 进行身份验证，以访问您的 Google Cloud 项目。

* 如果您使用 Colab 运行此笔记本，请使用下面的单元格并继续。
* 如果您使用 Vertex AI Workbench，请查看 [这里](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env) 的设置说明。

```python
# from google.colab import auth

# auth.authenticate_user()
```

### ☁ 设置您的 Google Cloud 项目
设置您的 Google Cloud 项目，以便您可以在此笔记本中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下操作：

* 运行 `gcloud config list`。
* 运行 `gcloud projects list`。
* 查看支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面填写您的 Google Cloud 项目 ID，然后运行该单元。

PROJECT_ID = "my-project-id"  # @param {type:"string"}

# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

## 基本用法

### 设置 Oracle 数据库连接
填写以下变量以提供您的 Oracle 数据库连接详细信息。

```python
# @title Set Your Values Here { display-mode: "form" }
HOST = "127.0.0.1"  # @param {type: "string"}
PORT = 3307  # @param {type: "integer"}
DATABASE = "my-database"  # @param {type: "string"}
TABLE_NAME = "message_store"  # @param {type: "string"}
USER = "my-user"  # @param {type: "string"}
PASSWORD = input("请输入用于数据库用户的密码: ")
```

如果您使用的是 `El Carro`，可以在 `El Carro` Kubernetes 实例的状态中找到主机名和端口值。使用您为 PDB 创建的用户密码。
示例

kubectl get -w instances.oracle.db.anthosapis.com -n db
NAME   DB ENGINE   VERSION   EDITION      ENDPOINT      URL                DB NAMES   BACKUP ID   READYSTATUS   READYREASON        DBREADYSTATUS   DBREADYREASON
mydb   Oracle      18c       Express      mydb-svc.db   34.71.69.25:6021                          False         CreateInProgress

### ElCarroEngine 连接池

`ElCarroEngine` 配置一个连接池到您的 Oracle 数据库，使您的应用程序能够成功连接，并遵循行业最佳实践。

```python
from langchain_google_el_carro import ElCarroEngine

elcarro_engine = ElCarroEngine.from_instance(
    db_host=HOST,
    db_port=PORT,
    db_name=DATABASE,
    db_user=USER,
    db_password=PASSWORD,
)
```

### 初始化表
`ElCarroChatMessageHistory` 类需要一个具有特定架构的数据库表以存储聊天消息历史记录。

`ElCarroEngine` 类有一个方法 `init_chat_history_table()`，可以用于为您创建具有正确架构的表。

```python
elcarro_engine.init_chat_history_table(table_name=TABLE_NAME)
```

### ElCarroChatMessageHistory

要初始化 `ElCarroChatMessageHistory` 类，您只需提供 3 个参数：

1. `elcarro_engine` - `ElCarroEngine` 引擎的实例。
1. `session_id` - 一个唯一标识符字符串，用于指定会话的 ID。
1. `table_name` : 存储聊天消息历史的 Oracle 数据库中的表名。


```python
from langchain_google_el_carro import ElCarroChatMessageHistory

history = ElCarroChatMessageHistory(
    elcarro_engine=elcarro_engine, session_id="test_session", table_name=TABLE_NAME
)
history.add_user_message("hi!")
history.add_ai_message("whats up?")
```


```python
history.messages
```

#### 清理
当特定会话的历史记录过时时，可以通过以下方式删除。

**注意：** 一旦删除，数据将不再存储在您的数据库中，且将永远消失。


```python
history.clear()
```

## 🔗 链接

我们可以轻松地将这个消息历史类与 [LCEL Runnables](/docs/how_to/message_history) 结合起来。

为此，我们将使用 [Google 的 Vertex AI 聊天模型](/docs/integrations/chat/google_vertex_ai_palm)，这要求您在 Google Cloud 项目中 [启用 Vertex AI API](https://console.cloud.google.com/flows/enableapi?apiid=aiplatform.googleapis.com)。

```python
# enable Vertex AI API
!gcloud services enable aiplatform.googleapis.com
```

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_google_vertexai import ChatVertexAI
```

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant."),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)

chain = prompt | ChatVertexAI(project=PROJECT_ID)
```

```python
chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: ElCarroChatMessageHistory(
        elcarro_engine,
        session_id=session_id,
        table_name=TABLE_NAME,
    ),
    input_messages_key="question",
    history_messages_key="history",
)
```

```python
# This is where we configure the session id
config = {"configurable": {"session_id": "test_session"}}
```

```python
chain_with_history.invoke({"question": "Hi! I'm bob"}, config=config)
```

```python
chain_with_history.invoke({"question": "Whats my name"}, config=config)
```