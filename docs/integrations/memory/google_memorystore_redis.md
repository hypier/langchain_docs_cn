---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/google_memorystore_redis.ipynb
---

# Google Memorystore for Redis

> [Google Cloud Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/memorystore-for-redis-overview) 是一个完全托管的服务，基于 Redis 内存数据存储构建应用缓存，提供亚毫秒级的数据访问。扩展您的数据库应用，利用 Memorystore for Redis 的 Langchain 集成构建 AI 驱动的体验。

本笔记本介绍如何使用 [Google Cloud Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/memorystore-for-redis-overview) 使用 `MemorystoreChatMessageHistory` 类存储聊天消息历史。

在 [GitHub](https://github.com/googleapis/langchain-google-memorystore-redis-python/) 上了解更多关于该包的信息。

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-memorystore-redis-python/blob/main/docs/chat_message_history.ipynb)

## 开始之前

要运行此笔记本，您需要执行以下操作：

* [创建一个 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)
* [启用 Memorystore for Redis API](https://console.cloud.google.com/flows/enableapi?apiid=redis.googleapis.com)
* [创建一个 Memorystore for Redis 实例](https://cloud.google.com/memorystore/docs/redis/create-instance-console)。确保版本大于或等于 5.0。

在确认可以访问此笔记本的运行时环境中的数据库后，填写以下值并在运行示例脚本之前运行该单元格。

```python
# @markdown 请指定与实例或演示目的相关联的端点。
ENDPOINT = "redis://127.0.0.1:6379"  # @param {type:"string"}
```

### 🦜🔗 库安装

集成位于其自己的 `langchain-google-memorystore-redis` 包中，因此我们需要安装它。


```python
%pip install -upgrade --quiet langchain-google-memorystore-redis
```

**仅限 Colab：** 取消注释以下单元以重新启动内核，或使用按钮重新启动内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重新启动终端。


```python
# # 自动在安装后重新启动内核，以便您的环境可以访问新包
# import IPython

# app = IPython.Application.instance()
# app.kernel.do_shutdown(True)
```

### ☁ 设置您的 Google Cloud 项目
设置您的 Google Cloud 项目，以便您可以在此笔记本中利用 Google Cloud 资源。

如果您不知道您的项目 ID，请尝试以下方法：

* 运行 `gcloud config list`。
* 运行 `gcloud projects list`。
* 查看支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
# @markdown 请在下面填写您的 Google Cloud 项目 ID，然后运行该单元格。

PROJECT_ID = "my-project-id"  # @param {type:"string"}

# 设置项目 ID
!gcloud config set project {PROJECT_ID}
```

### 🔐 认证
以登录此笔记本的 IAM 用户身份对 Google Cloud 进行身份验证，以访问您的 Google Cloud 项目。

* 如果您使用 Colab 运行此笔记本，请使用下面的单元并继续。
* 如果您使用 Vertex AI Workbench，请查看 [这里](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env) 的设置说明。


```python
from google.colab import auth

auth.authenticate_user()
```

## 基本用法

### MemorystoreChatMessageHistory

要初始化 `MemorystoreMessageHistory` 类，您只需提供两样东西：

1. `redis_client` - Memorystore Redis 的实例。
1. `session_id` - 每个聊天消息历史对象必须具有唯一的会话 ID。如果会话 ID 已经在 Redis 中存储了消息，则可以检索这些消息。


```python
import redis
from langchain_google_memorystore_redis import MemorystoreChatMessageHistory

# Connect to a Memorystore for Redis instance
redis_client = redis.from_url("redis://127.0.0.1:6379")

message_history = MemorystoreChatMessageHistory(redis_client, session_id="session1")
```


```python
message_history.messages
```

#### 清理

当特定会话的历史记录过时时，可以通过以下方式删除。

**注意：** 一旦删除，数据将不再存储在 Memorystore for Redis 中，并且将永远消失。


```python
message_history.clear()
```