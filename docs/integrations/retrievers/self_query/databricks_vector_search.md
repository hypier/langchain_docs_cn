---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/self_query/databricks_vector_search.ipynb
---

# Databricks 向量搜索

>[Databricks 向量搜索](https://docs.databricks.com/en/generative-ai/vector-search.html) 是一个无服务器相似性搜索引擎，允许您在向量数据库中存储数据的向量表示，包括元数据。使用向量搜索，您可以从由 Unity Catalog 管理的 Delta 表创建自动更新的向量搜索索引，并通过简单的 API 查询它们，以返回最相似的向量。

在本演示中，我们将展示如何使用 Databricks 向量搜索中的 `SelfQueryRetriever`。

## 创建 Databricks 向量存储索引
首先，我们需要创建一个 Databricks 向量存储索引，并用一些数据进行初始化。我们创建了一小组包含电影摘要的演示文档。

**注意：** 自查询检索器需要您安装 `lark`（`pip install lark`）以及特定于集成的要求。


```python
%pip install --upgrade --quiet  langchain-core databricks-vectorsearch langchain-openai tiktoken
```
```output
注意：您可能需要重启内核以使用更新的包。
```
我们想要使用 `OpenAIEmbeddings`，因此必须获取 OpenAI API 密钥。


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
databricks_host = getpass.getpass("Databricks host:")
databricks_token = getpass.getpass("Databricks token:")
```
```output
OpenAI API Key: ········
Databricks host: ········
Databricks token: ········
```

```python
from databricks.vector_search.client import VectorSearchClient
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
emb_dim = len(embeddings.embed_query("hello"))

vector_search_endpoint_name = "vector_search_demo_endpoint"


vsc = VectorSearchClient(
    workspace_url=databricks_host, personal_access_token=databricks_token
)
vsc.create_endpoint(name=vector_search_endpoint_name, endpoint_type="STANDARD")
```
```output
[NOTICE] 使用个人访问令牌 (PAT)。仅推荐用于开发。要提高性能，请使用基于服务主体的身份验证。要禁用此消息，请将 disable_notice=True 传递给 VectorSearchClient()。
```

```python
index_name = "udhay_demo.10x.demo_index"

index = vsc.create_direct_access_index(
    endpoint_name=vector_search_endpoint_name,
    index_name=index_name,
    primary_key="id",
    embedding_dimension=emb_dim,
    embedding_vector_column="text_vector",
    schema={
        "id": "string",
        "page_content": "string",
        "year": "int",
        "rating": "float",
        "genre": "string",
        "text_vector": "array<float>",
    },
)

index.describe()
```


```python
index = vsc.get_index(endpoint_name=vector_search_endpoint_name, index_name=index_name)

index.describe()
```


```python
from langchain_core.documents import Document

docs = [
    Document(
        page_content="一群科学家复活了恐龙，混乱随之而来",
        metadata={"id": 1, "year": 1993, "rating": 7.7, "genre": "动作"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在梦中，梦中又有梦...",
        metadata={"id": 2, "year": 2010, "genre": "惊悚", "rating": 8.2},
    ),
    Document(
        page_content="一群正常身材的女性极其善良，一些男性对她们心存向往",
        metadata={"id": 3, "year": 2019, "rating": 8.3, "genre": "剧情"},
    ),
    Document(
        page_content="三名男子走进区域，三名男子走出区域",
        metadata={"id": 4, "year": 1979, "rating": 9.9, "genre": "科幻"},
    ),
    Document(
        page_content="一名心理学家/侦探迷失在一系列梦中，梦中又有梦，且《盗梦空间》重复了这个想法",
        metadata={"id": 5, "year": 2006, "genre": "惊悚", "rating": 9.0},
    ),
    Document(
        page_content="玩具们活了过来，尽情享受",
        metadata={"id": 6, "year": 1995, "genre": "动画", "rating": 9.3},
    ),
]
```


```python
from langchain_community.vectorstores import DatabricksVectorSearch

vector_store = DatabricksVectorSearch(
    index,
    text_column="page_content",
    embedding=embeddings,
    columns=["year", "rating", "genre"],
)
```


```python
vector_store.add_documents(docs)
```

## 创建自查询检索器
现在我们可以实例化我们的检索器。为此，我们需要提前提供一些关于文档支持的元数据字段的信息，以及文档内容的简短描述。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import OpenAI

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
        name="rating", description="A 1-10 rating for the movie", type="float"
    ),
]
document_content_description = "Brief summary of a movie"
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vector_store, document_content_description, metadata_field_info, verbose=True
)
```

## 测试一下
现在我们可以尝试实际使用我们的检索器！



```python
# This example only specifies a relevant query
retriever.invoke("What are some movies about dinosaurs")
```



```output
[Document(page_content='A bunch of scientists bring back dinosaurs and mayhem breaks loose', metadata={'year': 1993.0, 'rating': 7.7, 'genre': 'action', 'id': 1.0}),
 Document(page_content='Toys come alive and have a blast doing so', metadata={'year': 1995.0, 'rating': 9.3, 'genre': 'animated', 'id': 6.0}),
 Document(page_content='Three men walk into the Zone, three men walk out of the Zone', metadata={'year': 1979.0, 'rating': 9.9, 'genre': 'science fiction', 'id': 4.0}),
 Document(page_content='A psychologist / detective gets lost in a series of dreams within dreams within dreams and Inception reused the idea', metadata={'year': 2006.0, 'rating': 9.0, 'genre': 'thriller', 'id': 5.0})]
```



```python
# This example specifies a filter
retriever.invoke("What are some highly rated movies (above 9)?")
```



```output
[Document(page_content='Toys come alive and have a blast doing so', metadata={'year': 1995.0, 'rating': 9.3, 'genre': 'animated', 'id': 6.0}),
 Document(page_content='Three men walk into the Zone, three men walk out of the Zone', metadata={'year': 1979.0, 'rating': 9.9, 'genre': 'science fiction', 'id': 4.0})]
```



```python
# This example specifies both a relevant query and a filter
retriever.invoke("What are the thriller movies that are highly rated?")
```



```output
[Document(page_content='A psychologist / detective gets lost in a series of dreams within dreams within dreams and Inception reused the idea', metadata={'year': 2006.0, 'rating': 9.0, 'genre': 'thriller', 'id': 5.0}),
 Document(page_content='Leo DiCaprio gets lost in a dream within a dream within a dream within a ...', metadata={'year': 2010.0, 'rating': 8.2, 'genre': 'thriller', 'id': 2.0})]
```



```python
# This example specifies a query and composite filter
retriever.invoke(
    "What's a movie after 1990 but before 2005 that's all about dinosaurs, \
    and preferably has a lot of action"
)
```



```output
[Document(page_content='A bunch of scientists bring back dinosaurs and mayhem breaks loose', metadata={'year': 1993.0, 'rating': 7.7, 'genre': 'action', 'id': 1.0})]
```

## Filter k

我们还可以使用自查询检索器来指定 `k`：要获取的文档数量。

我们可以通过将 `enable_limit=True` 传递给构造函数来实现这一点。

## 过滤 k

我们还可以使用自查询检索器来指定 `k`：要获取的文档数量。

我们可以通过将 `enable_limit=True` 传递给构造函数来实现这一点。


```python
retriever = SelfQueryRetriever.from_llm(
    llm,
    vector_store,
    document_content_description,
    metadata_field_info,
    verbose=True,
    enable_limit=True,
)
```


```python
retriever.invoke("What are two movies about dinosaurs?")
```