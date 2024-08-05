---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/tencentvectordb.ipynb
---

# 腾讯云 VectorDB

>[腾讯云 VectorDB](https://cloud.tencent.com/document/product/1709) 是一款完全托管、自主研发的企业级分布式数据库服务，旨在存储、检索和分析多维向量数据。该数据库支持多种索引类型和相似度计算方法。单个索引可以支持高达 10 亿的向量规模，并能够支持数百万的 QPS 和毫秒级的查询延迟。腾讯云向量数据库不仅可以为大型模型提供外部知识库，以提高大型模型响应的准确性，还可以广泛应用于推荐系统、自然语言处理服务、计算机视觉和智能客服等 AI 领域。

本笔记本演示了如何使用与腾讯向量数据库相关的功能。

要运行，您需要一个 [数据库实例。](https://cloud.tencent.com/document/product/1709/95101).

## 基本用法



```python
!pip3 install tcvectordb langchain-community
```


```python
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings.fake import FakeEmbeddings
from langchain_community.vectorstores import TencentVectorDB
from langchain_community.vectorstores.tencentvectordb import ConnectionParams
from langchain_text_splitters import CharacterTextSplitter
```

加载文档，将其拆分为多个块。


```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

我们支持两种嵌入文档的方式：
- 使用与 Langchain Embeddings 兼容的任何嵌入模型。
- 指定腾讯向量数据库的嵌入模型名称，选择如下：
    - `bge-base-zh`, 维度：768
    - `m3e-base`, 维度：768
    - `text2vec-large-chinese`, 维度：1024
    - `e5-large-v2`, 维度：1024
    - `multilingual-e5-base`, 维度：768 

以下代码展示了两种嵌入文档的方式，您可以通过注释掉其中一种来选择其中一种：


```python
##  您可以使用 Langchain Embeddings 模型，例如 OpenAIEmbeddings：

# from langchain_community.embeddings.openai import OpenAIEmbeddings
#
# embeddings = OpenAIEmbeddings()
# t_vdb_embedding = None

## 或者您可以使用腾讯嵌入模型，例如 `bge-base-zh`：

t_vdb_embedding = "bge-base-zh"  # bge-base-zh 是默认模型
embeddings = None
```

现在我们可以创建一个 TencentVectorDB 实例，您必须提供 `embeddings` 或 `t_vdb_embedding` 参数中的至少一个。如果两个参数都提供，则将使用 `embeddings` 参数：


```python
conn_params = ConnectionParams(
    url="http://10.0.X.X",
    key="eC4bLRy2va******************************",
    username="root",
    timeout=20,
)

vector_db = TencentVectorDB.from_documents(
    docs, embeddings, connection_params=conn_params, t_vdb_embedding=t_vdb_embedding
)
```


```python
query = "总统对凯坦吉·布朗·杰克逊说了什么"
docs = vector_db.similarity_search(query)
docs[0].page_content
```



```output
'今晚。我呼吁参议院：通过《投票自由法》。通过《约翰·刘易斯投票权法》。而且在此期间，通过《披露法》，让美国人知道谁在资助我们的选举。\n\n今晚，我想表彰一位为国家奉献一生的人：斯蒂芬·布雷耶大法官——一位退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。\n\n总统最严重的宪法责任之一是提名某人担任美国最高法院法官。\n\n四天前，我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官卓越的遗产。'
```



```python
vector_db = TencentVectorDB(embeddings, conn_params)

vector_db.add_texts(["安库什去了普林斯顿"])
query = "安库什去哪所大学？"
docs = vector_db.max_marginal_relevance_search(query)
docs[0].page_content
```



```output
'安库什去了普林斯顿'
```

## 元数据和过滤

腾讯 VectorDB 支持元数据和 [过滤](https://cloud.tencent.com/document/product/1709/95099#c6f6d3a3-02c5-4891-b0a1-30fe4daf18d8)。您可以向文档添加元数据，并根据元数据过滤搜索结果。

现在我们将创建一个新的 TencentVectorDB 集合，并演示如何根据元数据过滤搜索结果：

```python
from langchain_community.vectorstores.tencentvectordb import (
    META_FIELD_TYPE_STRING,
    META_FIELD_TYPE_UINT64,
    ConnectionParams,
    MetaField,
    TencentVectorDB,
)
from langchain_core.documents import Document

meta_fields = [
    MetaField(name="year", data_type=META_FIELD_TYPE_UINT64, index=True),
    MetaField(name="rating", data_type=META_FIELD_TYPE_STRING, index=False),
    MetaField(name="genre", data_type=META_FIELD_TYPE_STRING, index=True),
    MetaField(name="director", data_type=META_FIELD_TYPE_STRING, index=True),
]

docs = [
    Document(
        page_content="The Shawshank Redemption is a 1994 American drama film written and directed by Frank Darabont.",
        metadata={
            "year": 1994,
            "rating": "9.3",
            "genre": "drama",
            "director": "Frank Darabont",
        },
    ),
    Document(
        page_content="The Godfather is a 1972 American crime film directed by Francis Ford Coppola.",
        metadata={
            "year": 1972,
            "rating": "9.2",
            "genre": "crime",
            "director": "Francis Ford Coppola",
        },
    ),
    Document(
        page_content="The Dark Knight is a 2008 superhero film directed by Christopher Nolan.",
        metadata={
            "year": 2008,
            "rating": "9.0",
            "genre": "superhero",
            "director": "Christopher Nolan",
        },
    ),
    Document(
        page_content="Inception is a 2010 science fiction action film written and directed by Christopher Nolan.",
        metadata={
            "year": 2010,
            "rating": "8.8",
            "genre": "science fiction",
            "director": "Christopher Nolan",
        },
    ),
]

vector_db = TencentVectorDB.from_documents(
    docs,
    None,
    connection_params=ConnectionParams(
        url="http://10.0.X.X",
        key="eC4bLRy2va******************************",
        username="root",
        timeout=20,
    ),
    collection_name="movies",
    meta_fields=meta_fields,
)

query = "film about dream by Christopher Nolan"

# you can use the tencentvectordb filtering syntax with the `expr` parameter:
result = vector_db.similarity_search(query, expr='director="Christopher Nolan"')

# you can either use the langchain filtering syntax with the `filter` parameter:
# result = vector_db.similarity_search(query, filter='eq("director", "Christopher Nolan")')

result
```



```output
[Document(page_content='The Dark Knight is a 2008 superhero film directed by Christopher Nolan.', metadata={'year': 2008, 'rating': '9.0', 'genre': 'superhero', 'director': 'Christopher Nolan'}),
 Document(page_content='The Dark Knight is a 2008 superhero film directed by Christopher Nolan.', metadata={'year': 2008, 'rating': '9.0', 'genre': 'superhero', 'director': 'Christopher Nolan'}),
 Document(page_content='The Dark Knight is a 2008 superhero film directed by Christopher Nolan.', metadata={'year': 2008, 'rating': '9.0', 'genre': 'superhero', 'director': 'Christopher Nolan'}),
 Document(page_content='Inception is a 2010 science fiction action film written and directed by Christopher Nolan.', metadata={'year': 2010, 'rating': '8.8', 'genre': 'science fiction', 'director': 'Christopher Nolan'})]
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)