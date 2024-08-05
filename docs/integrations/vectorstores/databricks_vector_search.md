---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/databricks_vector_search.ipynb
---

# Databricks 向量搜索

Databricks 向量搜索是一个无服务器的相似性搜索引擎，允许您在向量数据库中存储数据的向量表示，包括元数据。通过向量搜索，您可以从由 Unity Catalog 管理的 Delta 表中创建自动更新的向量搜索索引，并使用简单的 API 查询它们以返回最相似的向量。

本笔记本演示了如何将 LangChain 与 Databricks 向量搜索结合使用。

安装本笔记本中使用的 `databricks-vectorsearch` 和相关的 Python 包。


```python
%pip install --upgrade --quiet  langchain-core databricks-vectorsearch langchain-openai tiktoken
```

使用 `OpenAIEmbeddings` 进行嵌入。


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

拆分文档并获取嵌入。


```python
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
emb_dim = len(embeddings.embed_query("hello"))
```

## 设置 Databricks 向量搜索客户端


```python
from databricks.vector_search.client import VectorSearchClient

vsc = VectorSearchClient()
```

## 创建向量搜索端点
此端点用于创建和访问向量搜索索引。


```python
vsc.create_endpoint(name="vector_search_demo_endpoint", endpoint_type="STANDARD")
```

## 创建直接向量访问索引
直接向量访问索引支持通过 REST API 或 SDK 直接读取和写入嵌入向量和元数据。对于此索引，您自己管理嵌入向量和索引更新。

```python
vector_search_endpoint_name = "vector_search_demo_endpoint"
index_name = "ml.llm.demo_index"

index = vsc.create_direct_access_index(
    endpoint_name=vector_search_endpoint_name,
    index_name=index_name,
    primary_key="id",
    embedding_dimension=emb_dim,
    embedding_vector_column="text_vector",
    schema={
        "id": "string",
        "text": "string",
        "text_vector": "array<float>",
        "source": "string",
    },
)

index.describe()
```

```python
from langchain_community.vectorstores import DatabricksVectorSearch

dvs = DatabricksVectorSearch(
    index, text_column="text", embedding=embeddings, columns=["source"]
)
```

## 将文档添加到索引


```python
dvs.add_documents(docs)
```

## 相似性搜索
可选的关键字参数用于 similarity_search，包括指定要检索的文档数量 k，以及用于基于 [此语法](https://docs.databricks.com/en/generative-ai/create-query-vector-search.html#use-filters-on-queries) 的元数据过滤的 filters 字典，以及可以是 ANN 或 HYBRID 的 [query_type](https://api-docs.databricks.com/python/vector-search/databricks.vector_search.html#databricks.vector_search.index.VectorSearchIndex.similarity_search) 


```python
query = "What did the president say about Ketanji Brown Jackson"
dvs.similarity_search(query)
print(docs[0].page_content)
```

## 使用 Delta Sync Index

您还可以使用 `DatabricksVectorSearch` 在 Delta Sync Index 中进行搜索。Delta Sync Index 会自动从 Delta 表中同步。您无需手动调用 `add_text`/`add_documents`。有关更多详细信息，请参见 [Databricks 文档页面](https://docs.databricks.com/en/generative-ai/vector-search.html#delta-sync-index-with-managed-embeddings)。

```python
dvs_delta_sync = DatabricksVectorSearch("catalog_name.schema_name.delta_sync_index")
dvs_delta_sync.similarity_search(query)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)