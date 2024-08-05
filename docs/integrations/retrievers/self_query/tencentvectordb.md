---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/self_query/tencentvectordb.ipynb
---

# 腾讯云 VectorDB

> [腾讯云 VectorDB](https://cloud.tencent.com/document/product/1709) 是一款完全托管、自主研发的企业级分布式数据库服务，旨在存储、检索和分析多维向量数据。

在本演示中，我们将展示如何使用腾讯云 VectorDB 的 `SelfQueryRetriever`。

## 创建一个 TencentVectorDB 实例
首先，我们需要创建一个 TencentVectorDB 并用一些数据进行初始化。我们创建了一小组包含电影摘要的演示文档。

**注意：** 自查询检索器需要安装 `lark`（`pip install lark`）以及特定集成的要求。

```python
%pip install --upgrade --quiet tcvectordb langchain-openai tiktoken lark
```
```output

[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m A new release of pip is available: [0m[31;49m23.2.1[0m[39;49m -> [0m[32;49m24.0[0m
[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m To update, run: [0m[32;49mpip install --upgrade pip[0m
注意：您可能需要重启内核以使用更新的包。
```
我们想使用 `OpenAIEmbeddings`，所以我们必须获取 OpenAI API 密钥。

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

创建一个 TencentVectorDB 实例并用一些数据进行初始化：

```python
from langchain_community.vectorstores.tencentvectordb import (
    ConnectionParams,
    MetaField,
    TencentVectorDB,
)
from langchain_core.documents import Document
from tcvectordb.model.enum import FieldType

meta_fields = [
    MetaField(name="year", data_type="uint64", index=True),
    MetaField(name="rating", data_type="string", index=False),
    MetaField(name="genre", data_type=FieldType.String, index=True),
    MetaField(name="director", data_type=FieldType.String, index=True),
]

docs = [
    Document(
        page_content="肖申克的救赎是一部1994年的美国剧情片，由弗兰克·达拉邦特编剧和导演。",
        metadata={
            "year": 1994,
            "rating": "9.3",
            "genre": "剧情",
            "director": "弗兰克·达拉邦特",
        },
    ),
    Document(
        page_content="教父是一部1972年的美国犯罪片，由弗朗西斯·福特·科波拉导演。",
        metadata={
            "year": 1972,
            "rating": "9.2",
            "genre": "犯罪",
            "director": "弗朗西斯·福特·科波拉",
        },
    ),
    Document(
        page_content="黑暗骑士是一部2008年的超级英雄电影，由克里斯托弗·诺兰导演。",
        metadata={
            "year": 2008,
            "rating": "9.0",
            "genre": "科幻",
            "director": "克里斯托弗·诺兰",
        },
    ),
    Document(
        page_content="盗梦空间是一部2010年的科幻动作片，由克里斯托弗·诺兰编剧和导演。",
        metadata={
            "year": 2010,
            "rating": "8.8",
            "genre": "科幻",
            "director": "克里斯托弗·诺兰",
        },
    ),
    Document(
        page_content="复仇者联盟是一部2012年的美国超级英雄电影，基于同名的漫威漫画超级英雄团队。",
        metadata={
            "year": 2012,
            "rating": "8.0",
            "genre": "科幻",
            "director": "乔斯·韦登",
        },
    ),
    Document(
        page_content="黑豹是一部2018年的美国超级英雄电影，基于同名的漫威漫画角色。",
        metadata={
            "year": 2018,
            "rating": "7.3",
            "genre": "科幻",
            "director": "瑞恩·库格勒",
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
    collection_name="self_query_movies",
    meta_fields=meta_fields,
    drop_old=True,
)
```

## 创建自查询检索器
现在我们可以实例化我们的检索器。为此，我们需要提前提供一些关于文档支持的元数据字段的信息，以及文档内容的简短描述。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import ChatOpenAI

metadata_field_info = [
    AttributeInfo(
        name="genre",
        description="The genre of the movie",
        type="string",
    ),
    AttributeInfo(
        name="year",
        description="The year the movie was released",
        type="integer",
    ),
    AttributeInfo(
        name="director",
        description="The name of the movie director",
        type="string",
    ),
    AttributeInfo(
        name="rating", description="A 1-10 rating for the movie", type="string"
    ),
]
document_content_description = "Brief summary of a movie"
```

```python
llm = ChatOpenAI(temperature=0, model="gpt-4", max_tokens=4069)
retriever = SelfQueryRetriever.from_llm(
    llm, vector_db, document_content_description, metadata_field_info, verbose=True
)
```

## 测试一下
现在我们可以尝试实际使用我们的检索器了！



```python
# This example only specifies a relevant query
retriever.invoke("movies about a superhero")
```



```output
[Document(page_content='The Dark Knight is a 2008 superhero film directed by Christopher Nolan.', metadata={'year': 2008, 'rating': '9.0', 'genre': 'science fiction', 'director': 'Christopher Nolan'}),
 Document(page_content='The Avengers is a 2012 American superhero film based on the Marvel Comics superhero team of the same name.', metadata={'year': 2012, 'rating': '8.0', 'genre': 'science fiction', 'director': 'Joss Whedon'}),
 Document(page_content='Black Panther is a 2018 American superhero film based on the Marvel Comics character of the same name.', metadata={'year': 2018, 'rating': '7.3', 'genre': 'science fiction', 'director': 'Ryan Coogler'}),
 Document(page_content='The Godfather is a 1972 American crime film directed by Francis Ford Coppola.', metadata={'year': 1972, 'rating': '9.2', 'genre': 'crime', 'director': 'Francis Ford Coppola'})]
```



```python
# This example only specifies a filter
retriever.invoke("movies that were released after 2010")
```



```output
[Document(page_content='The Avengers is a 2012 American superhero film based on the Marvel Comics superhero team of the same name.', metadata={'year': 2012, 'rating': '8.0', 'genre': 'science fiction', 'director': 'Joss Whedon'}),
 Document(page_content='Black Panther is a 2018 American superhero film based on the Marvel Comics character of the same name.', metadata={'year': 2018, 'rating': '7.3', 'genre': 'science fiction', 'director': 'Ryan Coogler'})]
```



```python
# This example specifies both a relevant query and a filter
retriever.invoke("movies about a superhero which were released after 2010")
```



```output
[Document(page_content='The Avengers is a 2012 American superhero film based on the Marvel Comics superhero team of the same name.', metadata={'year': 2012, 'rating': '8.0', 'genre': 'science fiction', 'director': 'Joss Whedon'}),
 Document(page_content='Black Panther is a 2018 American superhero film based on the Marvel Comics character of the same name.', metadata={'year': 2018, 'rating': '7.3', 'genre': 'science fiction', 'director': 'Ryan Coogler'})]
```

## 过滤 k

我们还可以使用自查询检索器来指定 `k`：要获取的文档数量。

我们可以通过将 `enable_limit=True` 传递给构造函数来实现这一点。


```python
retriever = SelfQueryRetriever.from_llm(
    llm,
    vector_db,
    document_content_description,
    metadata_field_info,
    verbose=True,
    enable_limit=True,
)
```


```python
retriever.invoke("what are two movies about a superhero")
```



```output
[Document(page_content='The Dark Knight is a 2008 superhero film directed by Christopher Nolan.', metadata={'year': 2008, 'rating': '9.0', 'genre': 'science fiction', 'director': 'Christopher Nolan'}),
 Document(page_content='The Avengers is a 2012 American superhero film based on the Marvel Comics superhero team of the same name.', metadata={'year': 2012, 'rating': '8.0', 'genre': 'science fiction', 'director': 'Joss Whedon'})]
```