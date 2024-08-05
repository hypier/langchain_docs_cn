---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/qdrant.ipynb
---

# Qdrant

>[Qdrant](https://qdrant.tech/documentation/)（读作 quadrant）是一个向量相似性搜索引擎。它提供了一个生产就绪的服务，具有方便的 API 来存储、搜索和管理向量，并支持附加负载和扩展过滤。这使得它在各种神经网络或基于语义的匹配、分面搜索及其他应用中非常有用。

本 документация 演示如何将 Qdrant 与 Langchain 一起用于密集/稀疏和混合检索。

> 本页面记录了 `QdrantVectorStore` 类，该类通过 Qdrant 的新 [Query API](https://qdrant.tech/blog/qdrant-1.10.x/) 支持多种检索模式。它要求您运行 Qdrant v1.10.0 或更高版本。

有多种运行 `Qdrant` 的模式，根据选择的不同，会有一些细微的差别。选项包括：
- 本地模式，无需服务器
- Docker 部署
- Qdrant Cloud

请参阅 [安装说明](https://qdrant.tech/documentation/install/)。


```python
%pip install langchain-qdrant langchain-openai langchain
```

我们将使用 `OpenAIEmbeddings` 进行演示。


```python
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_qdrant import QdrantVectorStore
from langchain_text_splitters import CharacterTextSplitter
```


```python
loader = TextLoader("some-file.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```

## 从 LangChain 连接到 Qdrant

### 本地模式

Python 客户端允许您在本地模式下运行相同的代码，而无需运行 Qdrant 服务器。这对于测试和调试或仅存储少量向量非常有用。嵌入可能完全保留在内存中或持久化到磁盘上。

#### 内存中

对于某些测试场景和快速实验，您可能更喜欢仅将所有数据保留在内存中，因此当客户端被销毁时数据会丢失——通常是在脚本/笔记本的末尾。

```python
qdrant = QdrantVectorStore.from_documents(
    docs,
    embeddings,
    location=":memory:",  # 仅使用内存存储的本地模式
    collection_name="my_documents",
)
```

#### 磁盘存储

在不使用 Qdrant 服务器的本地模式下，您还可以将向量存储在磁盘上，以便在运行之间持久化。

```python
qdrant = QdrantVectorStore.from_documents(
    docs,
    embeddings,
    path="/tmp/local_qdrant",
    collection_name="my_documents",
)
```

### 本地服务器部署

无论您选择使用 [Docker 容器](https://qdrant.tech/documentation/install/) 本地启动 Qdrant，还是选择使用 [官方 Helm 图表](https://github.com/qdrant/qdrant-helm) 进行 Kubernetes 部署，连接到该实例的方式都是相同的。您需要提供指向服务的 URL。

```python
url = "<---qdrant url here --->"
qdrant = QdrantVectorStore.from_documents(
    docs,
    embeddings,
    url=url,
    prefer_grpc=True,
    collection_name="my_documents",
)
```

### Qdrant Cloud

如果您不想忙于管理基础设施，可以选择在 [Qdrant Cloud](https://cloud.qdrant.io/) 上设置一个完全托管的 Qdrant 集群。包括一个永久免费 1GB 的集群供您试用。使用托管版本的 Qdrant 的主要区别在于，您需要提供一个 API 密钥，以保护您的部署不被公开访问。该值也可以在 `QDRANT_API_KEY` 环境变量中设置。


```python
url = "<---qdrant cloud cluster url here --->"
api_key = "<---api key here--->"
qdrant = QdrantVectorStore.from_documents(
    docs,
    embeddings,
    url=url,
    prefer_grpc=True,
    api_key=api_key,
    collection_name="my_documents",
)
```

## 使用现有集合

要获取 `langchain_qdrant.Qdrant` 的实例而不加载任何新文档或文本，可以使用 `Qdrant.from_existing_collection()` 方法。

```python
qdrant = QdrantVectorStore.from_existing_collection(
    embeddings=embeddings,
    collection_name="my_documents",
    url="http://localhost:6333",
)
```

## 重新创建集合

如果集合已经存在，则会重用该集合。将 `force_recreate` 设置为 `True` 可以删除旧集合并从头开始。

```python
url = "<---qdrant url here --->"
qdrant = QdrantVectorStore.from_documents(
    docs,
    embeddings,
    url=url,
    prefer_grpc=True,
    collection_name="my_documents",
    force_recreate=True,
)
```

## 相似性搜索

使用 Qdrant 向量存储的最简单场景是执行相似性搜索。在后台，我们的查询将被编码为向量嵌入，并用于在 Qdrant 集合中查找相似文档。

`QdrantVectorStore` 支持 3 种相似性搜索模式。可以在设置类时使用 `retrieval_mode` 参数进行配置。

- 密集向量搜索（默认）
- 稀疏向量搜索
- 混合搜索

### 密集向量搜索

要仅使用密集向量进行搜索，

- `retrieval_mode` 参数应设置为 `RetrievalMode.DENSE`（默认）。
- 应为 `embedding` 参数提供一个 [密集嵌入](https://python.langchain.com/v0.2/docs/integrations/text_embedding/) 值。


```python
from langchain_qdrant import RetrievalMode

qdrant = QdrantVectorStore.from_documents(
    docs,
    embedding=embeddings,
    location=":memory:",
    collection_name="my_documents",
    retrieval_mode=RetrievalMode.DENSE,
)

query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.similarity_search(query)
```

### 稀疏向量搜索

要仅使用稀疏向量进行搜索，

- `retrieval_mode` 参数应设置为 `RetrievalMode.SPARSE`。
- 必须提供一个实现 [`SparseEmbeddings`](https://github.com/langchain-ai/langchain/blob/master/libs/partners/qdrant/langchain_qdrant/sparse_embeddings.py) 接口的稀疏嵌入提供者作为 `sparse_embedding` 参数的值。

`langchain-qdrant` 包提供了基于 [FastEmbed](https://github.com/qdrant/fastembed) 的开箱即用实现。

要使用它，请安装 FastEmbed 包。

```python
%pip install fastembed
```

```python
from langchain_qdrant import FastEmbedSparse, RetrievalMode

sparse_embeddings = FastEmbedSparse(model_name="Qdrant/BM25")

qdrant = QdrantVectorStore.from_documents(
    docs,
    sparse_embedding=sparse_embeddings,
    location=":memory:",
    collection_name="my_documents",
    retrieval_mode=RetrievalMode.SPARSE,
)

query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.similarity_search(query)
```

### 混合向量搜索

要使用密集和稀疏向量进行混合搜索并进行得分融合，

- `retrieval_mode` 参数应设置为 `RetrievalMode.HYBRID`。
- 应为 `embedding` 参数提供一个 [dense embeddings](https://python.langchain.com/v0.2/docs/integrations/text_embedding/) 值。
- 必须提供一个使用任何稀疏嵌入提供者实现的 [`SparseEmbeddings`](https://github.com/langchain-ai/langchain/blob/master/libs/partners/qdrant/langchain_qdrant/sparse_embeddings.py) 接口的值给 `sparse_embedding` 参数。

请注意，如果您使用 `HYBRID` 模式添加了文档，则在搜索时可以切换到任何检索模式。因为密集和稀疏向量都可以在集合中使用。

```python
from langchain_qdrant import FastEmbedSparse, RetrievalMode

sparse_embeddings = FastEmbedSparse(model_name="Qdrant/BM25")

qdrant = QdrantVectorStore.from_documents(
    docs,
    embedding=embeddings,
    sparse_embedding=sparse_embeddings,
    location=":memory:",
    collection_name="my_documents",
    retrieval_mode=RetrievalMode.HYBRID,
)

query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.similarity_search(query)
```

## 具有评分的相似性搜索

有时我们可能希望执行搜索，同时获得相关性评分，以了解特定结果的好坏。 
返回的距离评分是余弦距离。因此，较低的评分更好。


```python
query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.similarity_search_with_score(query)
```


```python
document, score = found_docs[0]
print(document.page_content)
print(f"\nScore: {score}")
```

### 元数据过滤

Qdrant 拥有一个 [广泛的过滤系统](https://qdrant.tech/documentation/concepts/filtering/)，支持丰富的类型。还可以通过向 `similarity_search_with_score` 和 `similarity_search` 方法传递额外参数，在 Langchain 中使用这些过滤器。

```python
from qdrant_client.http import models

query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.similarity_search_with_score(query, filter=models.Filter(...))
```

## 最大边际相关性搜索 (MMR)

如果您想查找一些相似的文档，同时又希望获得多样化的结果，MMR 是您应该考虑的方法。最大边际相关性优化了与查询的相似性和所选文档之间的多样性。

请注意，MMR 搜索仅在您添加了 `DENSE` 或 `HYBRID` 模式的文档时可用。因为它需要密集向量。


```python
query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.max_marginal_relevance_search(query, k=2, fetch_k=10)
```


```python
for i, doc in enumerate(found_docs):
    print(f"{i + 1}.", doc.page_content, "\n")
```

## Qdrant 作为检索器

Qdrant 和其他向量存储一样，是一个 LangChain 检索器。 

```python
retriever = qdrant.as_retriever()
```

还可以指定使用 MMR 作为搜索策略，而不是相似性。

```python
retriever = qdrant.as_retriever(search_type="mmr")
```

```python
query = "What did the president say about Ketanji Brown Jackson"
retriever.invoke(query)[0]
```

## 自定义 Qdrant

在你的 Langchain 应用程序中，可以选择使用现有的 Qdrant 集合。在这种情况下，你可能需要定义如何将 Qdrant 点映射到 Langchain `Document`。

### 命名向量

Qdrant 支持通过命名向量 [每个点多个向量](https://qdrant.tech/documentation/concepts/collections/#collection-with-multiple-vectors)。如果您使用的是外部创建的集合或想要使用不同名称的向量，可以通过提供其名称进行配置。

```python
QdrantVectorStore.from_documents(
    docs,
    embedding=embeddings,
    sparse_embedding=sparse_embeddings,
    location=":memory:",
    collection_name="my_documents_2",
    retrieval_mode=RetrievalMode.HYBRID,
    vector_name="custom_vector",
    sparse_vector_name="custom_sparse_vector",
)
```

### 元数据

Qdrant 存储您的向量嵌入以及可选的类似 JSON 的有效负载。有效负载是可选的，但由于 LangChain 假设嵌入是从文档生成的，我们保留上下文数据，以便您可以提取原始文本。

默认情况下，您的文档将以以下有效负载结构存储：

```json
{
    "page_content": "Lorem ipsum dolor sit amet",
    "metadata": {
        "foo": "bar"
    }
}
```

不过，您可以决定为页面内容和元数据使用不同的键。如果您已经有一个希望重用的集合，这将很有用。

```python
QdrantVectorStore.from_documents(
    docs,
    embeddings,
    location=":memory:",
    collection_name="my_documents_2",
    content_payload_key="my_page_content_key",
    metadata_payload_key="my_meta",
)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)