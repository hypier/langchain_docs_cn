---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/google_memorystore_redis.ipynb
---

# Google Memorystore for Redis

> [Google Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/memorystore-for-redis-overview) 是一个完全托管的服务，基于 Redis 内存数据存储构建应用程序缓存，提供亚毫秒级的数据访问。扩展您的数据库应用程序，利用 Memorystore for Redis 的 Langchain 集成构建 AI 驱动的体验。

本笔记本介绍如何使用 [Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/memorystore-for-redis-overview) 通过 `MemorystoreDocumentLoader` 和 `MemorystoreDocumentSaver` [保存、加载和删除 langchain 文档](/docs/how_to#document-loaders)。

在 [GitHub](https://github.com/googleapis/langchain-google-memorystore-redis-python/) 上了解更多关于该包的信息。

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-memorystore-redis-python/blob/main/docs/document_loader.ipynb)

## 在开始之前

要运行此笔记本，您需要执行以下操作：

* [创建一个 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)
* [启用 Memorystore for Redis API](https://console.cloud.google.com/flows/enableapi?apiid=redis.googleapis.com)
* [创建一个 Memorystore for Redis 实例](https://cloud.google.com/memorystore/docs/redis/create-instance-console)。确保版本大于或等于 5.0。

在确认在此笔记本的运行时环境中可以访问数据库后，填写以下值并在运行示例脚本之前运行该单元格。

```python
# @markdown 请指定与实例相关联的端点和用于演示目的的键前缀。
ENDPOINT = "redis://127.0.0.1:6379"  # @param {type:"string"}
KEY_PREFIX = "doc:"  # @param {type:"string"}
```

### 🦜🔗 库安装

集成在其自己的 `langchain-google-memorystore-redis` 包中，因此我们需要安装它。

```python
%pip install -upgrade --quiet langchain-google-memorystore-redis
```

**仅限 Colab**：取消注释以下单元以重启内核，或使用按钮重启内核。对于 Vertex AI Workbench，您可以使用顶部的按钮重启终端。

```python
# # Automatically restart kernel after installs so that your environment can access the new packages
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

以在此笔记本中登录的 IAM 用户身份对 Google Cloud 进行身份验证，以访问您的 Google Cloud 项目。

- 如果您使用 Colab 运行此笔记本，请使用下面的单元并继续。
- 如果您使用 Vertex AI Workbench，请查看 [这里](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env) 的设置说明。


```python
from google.colab import auth

auth.authenticate_user()
```

## 基本用法

### 保存文档

使用 `MemorystoreDocumentSaver.add_documents(<documents>)` 保存 langchain 文档。要初始化 `MemorystoreDocumentSaver` 类，您需要提供两个参数：

1. `client` - 一个 `redis.Redis` 客户端对象。
1. `key_prefix` - 存储文档在 Redis 中的键前缀。

文档将存储在带有指定前缀 `key_prefix` 的随机生成的键中。或者，您可以通过在 `add_documents` 方法中指定 `ids` 来指定键的后缀。

```python
import redis
from langchain_core.documents import Document
from langchain_google_memorystore_redis import MemorystoreDocumentSaver

test_docs = [
    Document(
        page_content="Apple Granny Smith 150 0.99 1",
        metadata={"fruit_id": 1},
    ),
    Document(
        page_content="Banana Cavendish 200 0.59 0",
        metadata={"fruit_id": 2},
    ),
    Document(
        page_content="Orange Navel 80 1.29 1",
        metadata={"fruit_id": 3},
    ),
]
doc_ids = [f"{i}" for i in range(len(test_docs))]

redis_client = redis.from_url(ENDPOINT)
saver = MemorystoreDocumentSaver(
    client=redis_client,
    key_prefix=KEY_PREFIX,
    content_field="page_content",
)
saver.add_documents(test_docs, ids=doc_ids)
```

### 加载文档

初始化一个加载器，用于加载存储在具有特定前缀的Memorystore for Redis实例中的所有文档。

使用 `MemorystoreDocumentLoader.load()` 或 `MemorystoreDocumentLoader.lazy_load()` 加载 langchain 文档。`lazy_load` 返回一个生成器，该生成器在迭代过程中仅查询数据库。要初始化 `MemorystoreDocumentLoader` 类，您需要提供：

1. `client` - 一个 `redis.Redis` 客户端对象。
1. `key_prefix` - 用于在Redis中存储文档的键前缀。


```python
import redis
from langchain_google_memorystore_redis import MemorystoreDocumentLoader

redis_client = redis.from_url(ENDPOINT)
loader = MemorystoreDocumentLoader(
    client=redis_client,
    key_prefix=KEY_PREFIX,
    content_fields=set(["page_content"]),
)
for doc in loader.lazy_load():
    print("Loaded documents:", doc)
```

### 删除文档

使用 `MemorystoreDocumentSaver.delete()` 删除 Memorystore for Redis 实例中所有具有指定前缀的键。如果您知道键的后缀，也可以指定。

```python
docs = loader.load()
print("Documents before delete:", docs)

saver.delete(ids=[0])
print("Documents after delete:", loader.load())

saver.delete()
print("Documents after delete all:", loader.load())
```

## 高级用法

### 自定义文档页面内容和元数据

当使用多个内容字段初始化加载器时，加载文档的 `page_content` 将包含一个 JSON 编码的字符串，其顶层字段等于 `content_fields` 中指定的字段。

如果指定了 `metadata_fields`，加载文档的 `metadata` 字段将仅包含与指定的 `metadata_fields` 相等的顶层字段。如果任何元数据字段的值存储为 JSON 编码的字符串，则在加载到元数据字段之前将对其进行解码。

```python
loader = MemorystoreDocumentLoader(
    client=redis_client,
    key_prefix=KEY_PREFIX,
    content_fields=set(["content_field_1", "content_field_2"]),
    metadata_fields=set(["title", "author"]),
)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)