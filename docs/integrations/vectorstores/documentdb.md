---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/documentdb.ipynb
---

# Amazon Document DB

>[Amazon DocumentDB (与 MongoDB 兼容)](https://docs.aws.amazon.com/documentdb/) 使您可以轻松地在云中设置、操作和扩展与 MongoDB 兼容的数据库。
> 使用 Amazon DocumentDB，您可以运行相同的应用程序代码，并使用与 MongoDB 相同的驱动程序和工具。
> Amazon DocumentDB 的向量搜索结合了基于 JSON 的文档数据库的灵活性和丰富的查询能力，以及向量搜索的强大功能。


本笔记本向您展示如何使用 [Amazon Document DB 向量搜索](https://docs.aws.amazon.com/documentdb/latest/developerguide/vector-search.html) 将文档存储在集合中，创建索引并使用近似最近邻算法执行向量搜索查询，例如 "cosine"、"euclidean" 和 "dotProduct"。默认情况下，DocumentDB 创建层次可导航小世界 (HNSW) 索引。要了解其他支持的向量索引类型，请参考上述链接的文档。

要使用 DocumentDB，您必须首先部署一个集群。有关更多详细信息，请参阅 [开发者指南](https://docs.aws.amazon.com/documentdb/latest/developerguide/what-is.html)。

[注册](https://aws.amazon.com/free/) 免费开始使用。

```python
!pip install pymongo
```

```python
import getpass

# DocumentDB 连接字符串
# 即， "mongodb://{username}:{pass}@{cluster_endpoint}:{port}/?{params}"
CONNECTION_STRING = getpass.getpass("DocumentDB 集群 URI:")

INDEX_NAME = "izzy-test-index"
NAMESPACE = "izzy_test_db.izzy_test_collection"
DB_NAME, COLLECTION_NAME = NAMESPACE.split(".")
```

我们想使用 `OpenAIEmbeddings`，因此需要设置我们的 OpenAI 环境变量。

```python
import getpass
import os

# 设置 OpenAI 环境变量
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API 密钥:")
os.environ["OPENAI_EMBEDDINGS_DEPLOYMENT"] = (
    "smart-agent-embedding-ada"  # 嵌入模型的部署名称
)
os.environ["OPENAI_EMBEDDINGS_MODEL_NAME"] = "text-embedding-ada-002"  # 模型名称
```

现在，我们将加载文档到集合中，创建索引，然后对索引执行查询。

如果您对某些参数有疑问，请参考 [文档](https://docs.aws.amazon.com/documentdb/latest/developerguide/vector-search.html)。

```python
from langchain.vectorstores.documentdb import (
    DocumentDBSimilarityType,
    DocumentDBVectorSearch,
)
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

SOURCE_FILE_NAME = "../../how_to/state_of_the_union.txt"

loader = TextLoader(SOURCE_FILE_NAME)
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

# OpenAI 设置
model_deployment = os.getenv(
    "OPENAI_EMBEDDINGS_DEPLOYMENT", "smart-agent-embedding-ada"
)
model_name = os.getenv("OPENAI_EMBEDDINGS_MODEL_NAME", "text-embedding-ada-002")


openai_embeddings: OpenAIEmbeddings = OpenAIEmbeddings(
    deployment=model_deployment, model=model_name
)
```

```python
from pymongo import MongoClient

INDEX_NAME = "izzy-test-index-2"
NAMESPACE = "izzy_test_db.izzy_test_collection"
DB_NAME, COLLECTION_NAME = NAMESPACE.split(".")

client: MongoClient = MongoClient(CONNECTION_STRING)
collection = client[DB_NAME][COLLECTION_NAME]

model_deployment = os.getenv(
    "OPENAI_EMBEDDINGS_DEPLOYMENT", "smart-agent-embedding-ada"
)
model_name = os.getenv("OPENAI_EMBEDDINGS_MODEL_NAME", "text-embedding-ada-002")

vectorstore = DocumentDBVectorSearch.from_documents(
    documents=docs,
    embedding=openai_embeddings,
    collection=collection,
    index_name=INDEX_NAME,
)

# 上述模型使用的维度数量
dimensions = 1536

# 指定相似性算法，有效选项为：
#   cosine (COS), euclidean (EUC), dotProduct (DOT)
similarity_algorithm = DocumentDBSimilarityType.COS

vectorstore.create_index(dimensions, similarity_algorithm)
```

```output
{ 'createdCollectionAutomatically' : false,
   'numIndexesBefore' : 1,
   'numIndexesAfter' : 2,
   'ok' : 1,
   'operationTime' : Timestamp(1703656982, 1)}
```

```python
# 在查询的嵌入与文档的嵌入之间执行相似性搜索
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = vectorstore.similarity_search(query)
```

```python
print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯投票权法案》。在此期间，通过《披露法案》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：大法官斯蒂芬·布雷耶——一位退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最重要的宪法职责之一是提名某人担任美国最高法院法官。

我在四天前做了这件事，当时我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官卓越的遗产。
```
一旦文档加载完毕并创建了索引，您现在可以直接实例化向量存储并对索引执行查询。

```python
vectorstore = DocumentDBVectorSearch.from_connection_string(
    connection_string=CONNECTION_STRING,
    namespace=NAMESPACE,
    embedding=openai_embeddings,
    index_name=INDEX_NAME,
)

# 在查询与已摄取文档之间执行相似性搜索
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = vectorstore.similarity_search(query)
```

```python
print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯投票权法案》。在此期间，通过《披露法案》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：大法官斯蒂芬·布雷耶——一位退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最重要的宪法职责之一是提名某人担任美国最高法院法官。

我在四天前做了这件事，当时我提名了巡回上诉法院法官 Ketanji Brown Jackson。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官卓越的遗产。
```

```python
# 在查询与已摄取文档之间执行相似性搜索
query = "总统分享了哪些关于美国经济的统计数据"
docs = vectorstore.similarity_search(query)
```

```python
print(docs[0].page_content)
```
```output
与前一届政府通过的 2 万亿美元减税政策不同，该政策使美国顶层 1% 的人受益，《美国救援计划》帮助了工薪阶层——并且没有人被落下。

而且它奏效了。它创造了就业机会。大量的工作。

实际上——我们的经济去年创造了超过 650 万个新工作，是美国历史上在一年内创造的最多的工作。

我们的经济去年增长了 5.7%，是近 40 年以来最强劲的增长，迈出了为这个国家的工薪阶层带来根本变化的第一步。

在过去的 40 年里，我们被告知，如果我们给顶层减税，收益将会惠及其他所有人。

但这种涓滴经济理论导致了经济增长乏力、工资下降、赤字增加，以及近一个世纪以来顶层与其他人之间的差距最大。
```

## 问答

```python
qa_retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 25},
)
```

```python
from langchain_core.prompts import PromptTemplate

prompt_template = """使用以下上下文片段来回答最后的问题。如果你不知道答案，就说你不知道，不要试图编造答案。

{context}

问题: {question}
"""
PROMPT = PromptTemplate(
    template=prompt_template, input_variables=["context", "question"]
)
```

```python
from langchain.chains import RetrievalQA
from langchain_openai import OpenAI

qa = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",
    retriever=qa_retriever,
    return_source_documents=True,
    chain_type_kwargs={"prompt": PROMPT},
)

docs = qa({"query": "gpt-4 compute requirements"})

print(docs["result"])
print(docs["source_documents"])
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)