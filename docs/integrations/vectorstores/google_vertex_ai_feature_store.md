---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/google_vertex_ai_feature_store.ipynb
---

# Google Vertex AI 特征存储

> [Google Cloud Vertex 特征存储](https://cloud.google.com/vertex-ai/docs/featurestore/latest/overview) 通过让您以低延迟服务您的数据于 [Google Cloud BigQuery](https://cloud.google.com/bigquery?hl=en)，简化了您的 ML 特征管理和在线服务流程，包括执行嵌入的近似邻居检索的能力。

本教程向您展示如何直接从您的 BigQuery 数据中轻松执行低延迟的向量搜索和近似最近邻检索，从而以最小的设置启用强大的 ML 应用程序。我们将使用 `VertexFSVectorStore` 类来实现这一点。

该类是能够在 Google Cloud 中提供统一数据存储和灵活向量搜索的 2 个类的一部分：
- **BigQuery 向量搜索**：使用 `BigQueryVectorStore` 类，适合于无需基础设施设置的快速原型开发和批量检索。
- **特征存储在线存储**：使用 `VertexFSVectorStore` 类，支持手动或定期数据同步的低延迟检索。非常适合生产就绪的用户面向 GenAI 应用程序。

## 开始使用

### 安装库


```python
%pip install --upgrade --quiet  langchain langchain-google-vertexai "langchain-google-community[featurestore]"
```

要在此 Jupyter 运行时中使用新安装的包，您必须重启运行时。您可以通过运行下面的单元格来实现，这将重启当前的内核。


```python
import IPython

app = IPython.Application.instance()
app.kernel.do_shutdown(True)
```

## 在开始之前

#### 设置您的项目 ID

如果您不知道您的项目 ID，请尝试以下方法：
* 运行 `gcloud config list`。
* 运行 `gcloud projects list`。
* 查看支持页面：[定位项目 ID](https://support.google.com/googleapi/answer/7014113)。

```python
PROJECT_ID = ""  # @param {type:"string"}

# Set the project id
! gcloud config set project {PROJECT_ID}
```

#### 设置区域

您还可以更改 BigQuery 使用的 `REGION` 变量。了解更多关于 [BigQuery 区域](https://cloud.google.com/bigquery/docs/locations#supported_locations)。

```python
REGION = "us-central1"  # @param {type: "string"}
```

#### 设置数据集和表名

它们将是您的 BigQuery 向量存储。

```python
DATASET = "my_langchain_dataset"  # @param {type: "string"}
TABLE = "doc_and_vectors"  # @param {type: "string"}
```

### 验证您的笔记本环境

- 如果您使用 **Colab** 来运行此笔记本，请取消注释下面的单元格并继续。
- 如果您使用 **Vertex AI Workbench**，请查看 [这里](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env) 的设置说明。


```python
# from google.colab import auth as google_auth

# google_auth.authenticate_user()
```

## 演示：VertexFSVectorStore

### 创建嵌入类实例

您可能需要通过运行
`gcloud services enable aiplatform.googleapis.com --project {PROJECT_ID}`
（将 `{PROJECT_ID}` 替换为您的项目名称）来启用 Vertex AI API。

您可以使用任何 [LangChain 嵌入模型](/docs/integrations/text_embedding/)。


```python
from langchain_google_vertexai import VertexAIEmbeddings

embedding = VertexAIEmbeddings(
    model_name="textembedding-gecko@latest", project=PROJECT_ID
)
```

### 初始化 VertexFSVectorStore

如果 BigQuery 数据集和表不存在，将自动创建。有关所有可选参数，请参阅类定义 [这里](https://github.com/langchain-ai/langchain-google/blob/main/libs/community/langchain_google_community/bq_storage_vectorstores/featurestore.py#L33)。

```python
from langchain_google_community import VertexFSVectorStore

store = VertexFSVectorStore(
    project_id=PROJECT_ID,
    dataset_name=DATASET,
    table_name=TABLE,
    location=REGION,
    embedding=embedding,
)
```

### 添加文本

> 注意：第一次同步过程将花费大约 ~20 分钟，因为需要创建 Feature Online Store。

```python
all_texts = ["Apples and oranges", "Cars and airplanes", "Pineapple", "Train", "Banana"]
metadatas = [{"len": len(t)} for t in all_texts]

store.add_texts(all_texts, metadatas=metadatas)
```

您还可以通过执行 `sync_data` 方法按需启动同步。

```python
store.sync_data()
```

在生产环境中，您还可以使用 `cron_schedule` 类参数设置自动定时同步。
例如：
```python
store = VertexFSVectorStore(cron_schedule="TZ=America/Los_Angeles 00 13 11 8 *", ...)
```

### 搜索文档


```python
query = "I'd like a fruit."
docs = store.similarity_search(query)
print(docs)
```

### 按向量搜索文档


```python
query_vector = embedding.embed_query(query)
docs = store.similarity_search_by_vector(query_vector, k=2)
print(docs)
```

### 使用元数据过滤器搜索文档


```python
# This should only return "Banana" document.
docs = store.similarity_search_by_vector(query_vector, filter={"len": 6})
print(docs)
```

### 添加带嵌入的文本

您还可以使用`add_texts_with_embeddings`方法带上自己的嵌入。这对于可能需要在生成嵌入之前进行自定义预处理的多模态数据特别有用。

```python
items = ["some text"]
embs = embedding.embed(items)

ids = store.add_texts_with_embeddings(
    texts=["some text"], embs=embs, metadatas=[{"len": 1}]
)
```

### 使用 BigQuery 进行批量服务
您可以简单地使用方法 `.to_bq_vector_store()` 来获取一个 BigQueryVectorStore 对象，它为批量用例提供了优化的性能。所有必需的参数将自动从现有类中传递。有关您可以使用的所有参数，请参见 [类定义](https://github.com/langchain-ai/langchain-google/blob/main/libs/community/langchain_google_community/bq_storage_vectorstores/bigquery.py#L26)。

返回到 BigQueryVectorStore 同样简单，可以使用 `.to_vertex_fs_vector_store()` 方法。

```python
store.to_bq_vector_store()  # pass optional VertexFSVectorStore parameters as arguments
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)