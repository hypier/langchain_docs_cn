---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/surrealdb.ipynb
---

# SurrealDB

>[SurrealDB](https://surrealdb.com/) 是一个端到端的云原生数据库，旨在满足现代应用程序的需求，包括网页、移动端、无服务器、Jamstack、后端和传统应用程序。使用 SurrealDB，您可以简化数据库和 API 基础设施，减少开发时间，并快速且经济高效地构建安全、高性能的应用程序。
>
>**SurrealDB 的主要特点包括：**
>
>* **减少开发时间：** SurrealDB 通过消除大多数服务器端组件的需求来简化您的数据库和 API 堆栈，使您能够更快、更便宜地构建安全、高性能的应用程序。
>* **实时协作 API 后端服务：** SurrealDB 既充当数据库，又充当 API 后端服务，支持实时协作。
>* **支持多种查询语言：** SurrealDB 支持来自客户端设备的 SQL 查询、GraphQL、ACID 事务、WebSocket 连接、结构化和非结构化数据、图查询、全文索引以及地理空间查询。
>* **细粒度访问控制：** SurrealDB 提供基于行级权限的访问控制，使您能够精确管理数据访问。
>
>查看 [features](https://surrealdb.com/features)、最新 [releases](https://surrealdb.com/releases) 和 [documentation](https://surrealdb.com/docs)。

This notebook shows how to use functionality related to the `SurrealDBStore`.

## 设置

取消注释以下单元以安装 surrealdb。

```python
# %pip install --upgrade --quiet  surrealdb langchain langchain-community
```

## 使用 SurrealDBStore


```python
# add this import for running in jupyter notebook
import nest_asyncio

nest_asyncio.apply()
```


```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import SurrealDBStore
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```


```python
documents = TextLoader("../../how_to/state_of_the_union.txt").load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = HuggingFaceEmbeddings()
```

### 创建一个 SurrealDBStore 对象


```python
db = SurrealDBStore(
    dburl="ws://localhost:8000/rpc",  # 托管的 SurrealDB 数据库的 URL
    embedding_function=embeddings,
    db_user="root",  # 如果需要，SurrealDB 凭据：数据库用户名
    db_pass="root",  # 如果需要，SurrealDB 凭据：数据库密码
    # ns="langchain", # 用于向量存储的命名空间
    # db="database",  # 用于向量存储的数据库
    # collection="documents", # 用于向量存储的集合
)

# 这一步是初始化 SurrealDB 的底层异步库所需
await db.initialize()

# 从向量存储集合中删除所有现有文档
await db.adelete()

# 向向量存储中添加文档
ids = await db.aadd_documents(docs)

# 添加文档的文档 ID
ids[:5]
```



```output
['documents:38hz49bv1p58f5lrvrdc',
 'documents:niayw63vzwm2vcbh6w2s',
 'documents:it1fa3ktplbuye43n0ch',
 'documents:il8f7vgbbp9tywmsn98c',
 'documents:vza4c6cqje0avqd58gal']
```

### （可选）创建一个 SurrealDBStore 对象并添加文档

```python
await db.adelete()

db = await SurrealDBStore.afrom_documents(
    dburl="ws://localhost:8000/rpc",  # 托管的 SurrealDB 数据库的 URL
    embedding=embeddings,
    documents=docs,
    db_user="root",  # 如果需要，SurrealDB 凭据：数据库用户名
    db_pass="root",  # 如果需要，SurrealDB 凭据：数据库密码
    # ns="langchain", # 用于向量存储的命名空间
    # db="database",  # 用于向量存储的数据库
    # collection="documents", # 用于向量存储的集合
)
```

### 相似性搜索


```python
query = "总统对凯坦吉·布朗·杰克逊说了什么"
docs = await db.asimilarity_search(query)
```


```python
print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《投票自由法》。通过《约翰·刘易斯投票权法》。同时通过《披露法》，让美国人知道是谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：史蒂芬·布雷耶法官——一位退伍军人、宪法学者，以及即将退休的美国最高法院法官。布雷耶法官，感谢您的服务。

总统最重要的宪法责任之一就是提名某人担任美国最高法院法官。

四天前，我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律思想家之一，将继续布雷耶法官卓越的遗产。
```

### 相似性搜索与得分

返回的距离得分为余弦距离。因此，得分越低越好。


```python
docs = await db.asimilarity_search_with_score(query)
```


```python
docs[0]
```



```output
(Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'id': 'documents:slgdlhjkfknhqo15xz0w', 'source': '../../how_to/state_of_the_union.txt'}),
 0.39839531721941895)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)