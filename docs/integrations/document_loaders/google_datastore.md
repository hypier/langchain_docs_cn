---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/google_datastore.ipynb
---

# Google Firestore 在 Datastore 模式下

> [Firestore 在 Datastore 模式下](https://cloud.google.com/datastore) 是一个为自动扩展、高性能和简化应用开发而构建的 NoSQL 文档数据库。扩展您的数据库应用，以利用 Datastore 的 Langchain 集成构建 AI 驱动的体验。

本笔记本介绍如何使用 [Firestore 在 Datastore 模式下](https://cloud.google.com/datastore) 使用 `DatastoreLoader` 和 `DatastoreSaver` 来 [保存、加载和删除 langchain 文档](/docs/how_to#document-loaders)。

在 [GitHub](https://github.com/googleapis/langchain-google-datastore-python/) 上了解更多关于该包的信息。

[![在 Colab 中打开](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/googleapis/langchain-google-datastore-python/blob/main/docs/document_loader.ipynb)

## 开始之前

要运行此笔记本，您需要执行以下操作：

* [创建一个 Google Cloud 项目](https://developers.google.com/workspace/guides/create-project)
* [启用 Datastore API](https://console.cloud.google.com/flows/enableapi?apiid=datastore.googleapis.com)
* [创建一个 Firestore in Datastore Mode 数据库](https://cloud.google.com/datastore/docs/manage-databases)

在确认可以访问此笔记本的运行时环境中的数据库后，请填写以下值并在运行示例脚本之前运行该单元。

### 🦜🔗 库安装

集成位于其自己的 `langchain-google-datastore` 包中，因此我们需要安装它。

```python
%pip install -upgrade --quiet langchain-google-datastore
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

### 🔐 身份验证

作为已登录此笔记本的 IAM 用户，进行 Google Cloud 身份验证以访问您的 Google Cloud 项目。

- 如果您使用 Colab 运行此笔记本，请使用下面的单元并继续。
- 如果您使用 Vertex AI Workbench，请查看 [此处](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env) 的设置说明。


```python
from google.colab import auth

auth.authenticate_user()
```

## 基本用法

### 保存文档

使用 `DatastoreSaver.upsert_documents(<documents>)` 保存 langchain 文档。默认情况下，它会尝试从文档元数据中的 `key` 提取实体键。

```python
from langchain_core.documents import Document
from langchain_google_datastore import DatastoreSaver

saver = DatastoreSaver()

data = [Document(page_content="Hello, World!")]
saver.upsert_documents(data)
```

#### 无键保存文档

如果指定了 `kind`，文档将使用自动生成的 ID 存储。

```python
saver = DatastoreSaver("MyKind")

saver.upsert_documents(data)
```

### 通过 Kind 加载文档

使用 `DatastoreLoader.load()` 或 `DatastoreLoader.lazy_load()` 加载 langchain 文档。`lazy_load` 返回一个生成器，该生成器在迭代期间仅查询数据库。要初始化 `DatastoreLoader` 类，您需要提供：
1. `source` - 加载文档的来源。它可以是 Query 的实例或要读取的 Datastore kind 的名称。

```python
from langchain_google_datastore import DatastoreLoader

loader = DatastoreLoader("MyKind")
data = loader.load()
```

### 通过查询加载文档

除了从种类加载文档外，我们还可以选择通过查询加载文档。例如：

```python
from google.cloud import datastore

client = datastore.Client(database="non-default-db", namespace="custom_namespace")
query_load = client.query(kind="MyKind")
query_load.add_filter("region", "=", "west_coast")

loader_document = DatastoreLoader(query_load)

data = loader_document.load()
```

### 删除文档

使用 `DatastoreSaver.delete_documents(<documents>)` 从数据存储中删除一组 langchain 文档。


```python
saver = DatastoreSaver()

saver.delete_documents(data)

keys_to_delete = [
    ["Kind1", "identifier"],
    ["Kind2", 123],
    ["Kind3", "identifier", "NestedKind", 456],
]
# 文档将被忽略，仅使用文档 ID。
saver.delete_documents(data, keys_to_delete)
```

## 高级用法

### 加载具有自定义文档页面内容和元数据的文档

`page_content_properties` 和 `metadata_properties` 的参数将指定要写入 LangChain 文档 `page_content` 和 `metadata` 的实体属性。

```python
loader = DatastoreLoader(
    source="MyKind",
    page_content_fields=["data_field"],
    metadata_fields=["metadata_field"],
)

data = loader.load()
```

### 自定义页面内容格式

当 `page_content` 仅包含一个字段时，信息将仅为该字段的值。否则，`page_content` 将采用 JSON 格式。

### 自定义连接与身份验证


```python
from google.auth import compute_engine
from google.cloud.firestore import Client

client = Client(database="non-default-db", creds=compute_engine.Credentials())
loader = DatastoreLoader(
    source="foo",
    client=client,
)
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)