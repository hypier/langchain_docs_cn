---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/milvus.ipynb
---

# Milvus

>[Milvus](https://milvus.io/docs/overview.md) 是一个数据库，用于存储、索引和管理由深度神经网络和其他机器学习（ML）模型生成的大规模嵌入向量。

本笔记本展示了如何使用与 Milvus 向量数据库相关的功能。

您需要使用 `pip install -qU langchain-milvus` 安装 `langchain-milvus` 以使用此集成。



```python
%pip install --upgrade --quiet  langchain_milvus
```

最新版本的 pymilvus 附带了一个本地向量数据库 Milvus Lite，适合原型开发。如果您有大规模数据，例如超过一百万个文档，我们建议在 [docker 或 kubernetes](https://milvus.io/docs/install_standalone-docker.md#Start-Milvus) 上设置一个更高性能的 Milvus 服务器。

我们想使用 OpenAIEmbeddings，因此我们必须获取 OpenAI API 密钥。


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```


```python
from langchain_community.document_loaders import TextLoader
from langchain_milvus.vectorstores import Milvus
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```


```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```


```python
# 最简单的方法是使用 Milvus Lite，所有内容都存储在本地文件中。
# 如果您有 Milvus 服务器，可以使用服务器 URI，例如 "http://localhost:19530"。
URI = "./milvus_demo.db"

vector_db = Milvus.from_documents(
    docs,
    embeddings,
    connection_args={"uri": URI},
)
```


```python
query = "What did the president say about Ketanji Brown Jackson"
docs = vector_db.similarity_search(query)
```


```python
docs[0].page_content
```



```output
'Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.'
```

### 使用 Milvus Collections 对数据进行分类

您可以在同一个 Milvus 实例中将不同的无关文档存储在不同的集合中，以保持上下文。

以下是创建新集合的方法


```python
vector_db = Milvus.from_documents(
    docs,
    embeddings,
    collection_name="collection_1",
    connection_args={"uri": URI},
)
```

以下是检索已存储集合的方法


```python
vector_db = Milvus(
    embeddings,
    connection_args={"uri": URI},
    collection_name="collection_1",
)
```

检索后，您可以像往常一样继续查询它。

### 按用户检索

在构建检索应用时，您通常需要考虑多个用户。这意味着您可能不仅为一个用户存储数据，而是为许多不同的用户存储数据，并且他们不应该能够看到彼此的数据。

Milvus 推荐使用 [partition_key](https://milvus.io/docs/multi_tenancy.md#Partition-key-based-multi-tenancy) 来实现多租户，这里是一个示例。
> Partition key 的功能在 Milvus Lite 中目前不可用，如果您想使用它，您需要从 [docker 或 kubernetes](https://milvus.io/docs/install_standalone-docker.md#Start-Milvus) 启动 Milvus 服务器。

```python
from langchain_core.documents import Document

docs = [
    Document(page_content="i worked at kensho", metadata={"namespace": "harrison"}),
    Document(page_content="i worked at facebook", metadata={"namespace": "ankush"}),
]
vectorstore = Milvus.from_documents(
    docs,
    embeddings,
    connection_args={"uri": URI},
    drop_old=True,
    partition_key_field="namespace",  # 使用 "namespace" 字段作为分区键
)
```

要使用分区键进行搜索，您应该在搜索请求的布尔表达式中包含以下任一项：

`search_kwargs={"expr": '<partition_key> == "xxxx"'}`

`search_kwargs={"expr": '<partition_key> == in ["xxx", "xxx"]'}`

请将 `<partition_key>` 替换为指定为分区键的字段名称。

Milvus 根据指定的分区键进行分区，按照分区键过滤实体，并在过滤后的实体中进行搜索。

```python
# 这将仅获取 Ankush 的文档
vectorstore.as_retriever(search_kwargs={"expr": 'namespace == "ankush"'}).invoke(
    "where did i work?"
)
```

```output
[Document(page_content='i worked at facebook', metadata={'namespace': 'ankush'})]
```

```python
# 这将仅获取 Harrison 的文档
vectorstore.as_retriever(search_kwargs={"expr": 'namespace == "harrison"'}).invoke(
    "where did i work?"
)
```

```output
[Document(page_content='i worked at kensho', metadata={'namespace': 'harrison'})]
```

### 删除或更新插入（upsert）一个或多个实体


```python
from langchain_core.documents import Document

# Insert data sample
docs = [
    Document(page_content="foo", metadata={"id": 1}),
    Document(page_content="bar", metadata={"id": 2}),
    Document(page_content="baz", metadata={"id": 3}),
]
vector_db = Milvus.from_documents(
    docs,
    embeddings,
    connection_args={"uri": URI},
)

# Search pks (primary keys) using expression
expr = "id in [1,2]"
pks = vector_db.get_pks(expr)

# Delete entities by pks
result = vector_db.delete(pks)

# Upsert (Update/Insert)
new_docs = [
    Document(page_content="new_foo", metadata={"id": 1}),
    Document(page_content="new_bar", metadata={"id": 2}),
    Document(page_content="upserted_bak", metadata={"id": 3}),
]
upserted_pks = vector_db.upsert(pks, new_docs)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)