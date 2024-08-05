---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/google_bigquery_vector_search.ipynb
---

# Google BigQuery 向量搜索

> [Google Cloud BigQuery 向量搜索](https://cloud.google.com/bigquery/docs/vector-search-intro) 让您使用 GoogleSQL 进行语义搜索，使用向量索引以快速获取近似结果，或使用暴力搜索以获取精确结果。

本教程演示如何在 LangChain 中使用端到端的数据和嵌入管理系统，并提供在 BigQuery 中使用 `BigQueryVectorStore` 类的可扩展语义搜索。此类是能够提供统一数据存储和灵活向量搜索的一组 2 个类的一部分：
- **BigQuery 向量搜索**：使用 `BigQueryVectorStore` 类，适合快速原型开发，无需基础设施设置和批量检索。
- **特征存储在线存储**：使用 `VertexFSVectorStore` 类，支持手动或定期的数据同步，适合面向用户的 GenAI 生产就绪应用程序。

## 开始使用

### 安装库


```python
%pip install --upgrade --quiet  langchain langchain-google-vertexai "langchain-google-community[featurestore]"
```

要在此 Jupyter 运行时中使用新安装的包，您必须重启运行时。您可以通过运行下面的单元格来实现，这将重启当前内核。


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
* 查看支持页面：[查找项目 ID](https://support.google.com/googleapi/answer/7014113)。

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

- 如果您正在使用 **Colab** 来运行此笔记本，请取消注释下面的单元格并继续。
- 如果您正在使用 **Vertex AI Workbench**，请查看[这里](https://github.com/GoogleCloudPlatform/generative-ai/tree/main/setup-env)的设置说明。


```python
# from google.colab import auth as google_auth

# google_auth.authenticate_user()
```

## 演示：BigQueryVectorStore

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

### 初始化 BigQueryVectorStore

如果 BigQuery 数据集和表不存在，将自动创建。有关所有可选参数的类定义，请参见 [这里](https://github.com/langchain-ai/langchain-google/blob/main/libs/community/langchain_google_community/bq_storage_vectorstores/bigquery.py#L26)。

```python
from langchain_google_community import BigQueryVectorStore

store = BigQueryVectorStore(
    project_id=PROJECT_ID,
    dataset_name=DATASET,
    table_name=TABLE,
    location=REGION,
    embedding=embedding,
)
```

### 添加文本


```python
all_texts = ["Apples and oranges", "Cars and airplanes", "Pineapple", "Train", "Banana"]
metadatas = [{"len": len(t)} for t in all_texts]

store.add_texts(all_texts, metadatas=metadatas)
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

### 批量搜索
BigQueryVectorStore 提供了一个 `batch_search` 方法用于可扩展的向量相似性搜索。



```python
results = store.batch_search(
    embeddings=None,  # can pass embeddings or
    queries=["search_query", "search_query"],  # can pass queries
)
```

### 添加带嵌入的文本

您还可以使用 `add_texts_with_embeddings` 方法添加自己的嵌入。这对于可能需要在生成嵌入之前进行自定义预处理的多模态数据特别有用。

```python
items = ["some text"]
embs = embedding.embed(items)

ids = store.add_texts_with_embeddings(
    texts=["some text"], embs=embs, metadatas=[{"len": 1}]
)
```

### 低延迟服务与特征存储
您可以简单地使用方法 `.to_vertex_fs_vector_store()` 来获取一个 VertexFSVectorStore 对象，该对象为在线用例提供低延迟。所有必需的参数将自动从现有的 BigQueryVectorStore 类中转移。有关您可以使用的所有其他参数，请参见 [类定义](https://github.com/langchain-ai/langchain-google/blob/main/libs/community/langchain_google_community/bq_storage_vectorstores/featurestore.py#L33)。

返回到 BigQueryVectorStore 同样简单，使用 `.to_bq_vector_store()` 方法即可。

```python
store.to_vertex_fs_vector_store()  # pass optional VertexFSVectorStore parameters as arguments
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)