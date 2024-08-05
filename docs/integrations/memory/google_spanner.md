---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/google_spanner.ipynb
---

# Google Spanner
> [Google Cloud Spanner](https://cloud.google.com/spanner) 是一个高度可扩展的数据库，它将无限的可扩展性与关系语义相结合，例如二级索引、强一致性、模式和 SQL，提供 99.999% 的可用性，提供一个简单的解决方案。

本笔记本介绍如何使用 `Spanner` 来存储聊天消息历史记录，使用 `SpannerChatMessageHistory` 类。
在 [GitHub](https://github.com/googleapis/langchain-google-spanner-python/) 上了解更多关于该包的信息。

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-spanner-python/blob/main/samples/chat_message_history.ipynb)

## 开始之前

要运行此笔记本，您需要执行以下操作：

 * [创建 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)
 * [启用 Cloud Spanner API](https://console.cloud.google.com/flows/enableapi?apiid=spanner.googleapis.com)
 * [创建 Spanner 实例](https://cloud.google.com/spanner/docs/create-manage-instances)
 * [创建 Spanner 数据库](https://cloud.google.com/spanner/docs/create-manage-databases)

### 🦜🔗 库安装
集成在其自己的 `langchain-google-spanner` 包中，因此我们需要安装它。

```python
%pip install --upgrade --quiet langchain-google-spanner
```

**仅限 Colab:** 取消注释以下单元以重启内核，或使用按钮重启内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重启终端。

```python
# # Automatically restart kernel after installs so that your environment can access the new packages
# import IPython

# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### 🔐 身份验证
以登录此笔记本的 IAM 用户身份对 Google Cloud 进行身份验证，以便访问您的 Google Cloud 项目。

* 如果您使用 Colab 来运行此笔记本，请使用下面的单元并继续。
* 如果您使用 Vertex AI Workbench，请查看 [此处](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env) 的设置说明。


```python
from google.colab import auth

auth.authenticate_user()
```

### ☁ 设置您的 Google Cloud 项目
设置您的 Google Cloud 项目，以便您可以在此笔记本中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下方法：

* 运行 `gcloud config list`。
* 运行 `gcloud projects list`。
* 查看支持页面：[找到项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面填写您的 Google Cloud 项目 ID，然后运行该单元格。

PROJECT_ID = "my-project-id"  # @param {type:"string"}

# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

### 💡 API启用
`langchain-google-spanner`包要求您在Google Cloud项目中[启用Spanner API](https://console.cloud.google.com/flows/enableapi?apiid=spanner.googleapis.com)。

```python
# enable Spanner API
!gcloud services enable spanner.googleapis.com
```

## 基本用法

### 设置 Spanner 数据库值
在 [Spanner 实例页面](https://console.cloud.google.com/spanner) 找到您的数据库值。

```python
# @title Set Your Values Here { display-mode: "form" }
INSTANCE = "my-instance"  # @param {type: "string"}
DATABASE = "my-database"  # @param {type: "string"}
TABLE_NAME = "message_store"  # @param {type: "string"}
```

### 初始化表
`SpannerChatMessageHistory` 类需要一个具有特定架构的数据库表来存储聊天消息历史。

辅助方法 `init_chat_history_table()` 可用于为您创建具有正确架构的表。


```python
from langchain_google_spanner import (
    SpannerChatMessageHistory,
)

SpannerChatMessageHistory.init_chat_history_table(table_name=TABLE_NAME)
```

### SpannerChatMessageHistory

要初始化 `SpannerChatMessageHistory` 类，您只需提供 3 个参数：

1. `instance_id` - Spanner 实例的名称
1. `database_id` - Spanner 数据库的名称
1. `session_id` - 一个唯一标识符字符串，用于指定会话的 ID。
1. `table_name` - 用于存储聊天消息历史记录的数据库表的名称。


```python
message_history = SpannerChatMessageHistory(
    instance_id=INSTANCE,
    database_id=DATABASE,
    table_name=TABLE_NAME,
    session_id="user-session-id",
)

message_history.add_user_message("hi!")
message_history.add_ai_message("whats up?")
```


```python
message_history.messages
```

## 自定义客户端
默认创建的客户端是默认客户端。要使用非默认客户端，可以将 [自定义客户端](https://cloud.google.com/spanner/docs/samples/spanner-create-client-with-query-options#spanner_create_client_with_query_options-python) 传递给构造函数。

```python
from google.cloud import spanner

custom_client_message_history = SpannerChatMessageHistory(
    instance_id="my-instance",
    database_id="my-database",
    client=spanner.Client(...),
)
```

## 清理

当特定会话的历史记录过时时，可以通过以下方式删除。  
注意：一旦删除，数据将不再存储在 Cloud Spanner 中，并且将永久丢失。

```python
message_history = SpannerChatMessageHistory(
    instance_id=INSTANCE,
    database_id=DATABASE,
    table_name=TABLE_NAME,
    session_id="user-session-id",
)

message_history.clear()
```