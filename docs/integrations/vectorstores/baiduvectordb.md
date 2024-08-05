---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/baiduvectordb.ipynb
---

# 百度 VectorDB

>[Baidu VectorDB](https://cloud.baidu.com/product/vdb.html) 是一款强大、企业级的分布式数据库服务，由百度智能云精心开发并全面管理。它以卓越的多维向量数据存储、检索和分析能力而脱颖而出。VectorDB 的核心基于百度自主研发的“Mochow”向量数据库内核，确保高性能、高可用性和安全性，同时具备出色的可扩展性和用户友好性。

>该数据库服务支持多种索引类型和相似度计算方法，满足各种使用场景。VectorDB 的一个突出特点是能够管理高达 100 亿的巨大向量规模，同时保持令人印象深刻的查询性能，支持每秒数百万次查询（QPS），并具有毫秒级的查询延迟。

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 才能使用此集成。

本笔记本展示了如何使用与百度 VectorDB 相关的功能。

要运行，您需要一个 [数据库实例。](https://cloud.baidu.com/doc/VDB/s/hlrsoazuf)


```python
!pip3 install pymochow
```


```python
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings.fake import FakeEmbeddings
from langchain_community.vectorstores import BaiduVectorDB
from langchain_community.vectorstores.baiduvectordb import ConnectionParams
from langchain_text_splitters import CharacterTextSplitter
```


```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = FakeEmbeddings(size=128)
```


```python
conn_params = ConnectionParams(
    endpoint="http://192.168.xx.xx:xxxx", account="root", api_key="****"
)

vector_db = BaiduVectorDB.from_documents(
    docs, embeddings, connection_params=conn_params, drop_old=True
)
```


```python
query = "What did the president say about Ketanji Brown Jackson"
docs = vector_db.similarity_search(query)
docs[0].page_content
```


```python
vector_db = BaiduVectorDB(embeddings, conn_params)
vector_db.add_texts(["Ankush went to Princeton"])
query = "Where did Ankush go to college?"
docs = vector_db.max_marginal_relevance_search(query)
docs[0].page_content
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)