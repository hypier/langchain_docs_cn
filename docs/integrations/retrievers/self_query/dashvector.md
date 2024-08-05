---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/self_query/dashvector.ipynb
---

# DashVector

> [DashVector](https://help.aliyun.com/document_detail/2510225.html) 是一个完全托管的向量数据库服务，支持高维稠密和稀疏向量、实时插入和过滤搜索。它能够自动扩展，并能适应不同的应用需求。
> 向量检索服务 `DashVector` 基于 `DAMO Academy` 独立开发的高效向量引擎 `Proxima` 核心，
> 提供具有水平扩展能力的云原生、完全托管的向量检索服务。
> `DashVector` 通过简单易用的 SDK/API 接口，展示了其强大的向量管理、向量查询及其他多样化能力，可以被上层 AI 应用快速集成，从而提供包括大模型生态、多模态 AI 搜索、分子结构分析等多种应用场景所需的高效向量检索能力。

在本笔记本中，我们将演示与 `DashVector` 向量存储一起使用的 `SelfQueryRetriever`。

## 创建 DashVector 向量存储

首先，我们需要创建一个 `DashVector` 向量存储，并用一些数据进行初始化。我们创建了一小组包含电影摘要的示例文档。

要使用 DashVector，您必须安装 `dashvector` 包，并且必须拥有 API 密钥和环境。以下是 [安装说明](https://help.aliyun.com/document_detail/2510223.html)。

注意：自查询检索器需要您安装 `lark` 包。


```python
%pip install --upgrade --quiet  lark dashvector
```


```python
import os

import dashvector

client = dashvector.Client(api_key=os.environ["DASHVECTOR_API_KEY"])
```


```python
from langchain_community.embeddings import DashScopeEmbeddings
from langchain_community.vectorstores import DashVector
from langchain_core.documents import Document

embeddings = DashScopeEmbeddings()

# 创建 DashVector 集合
client.create("langchain-self-retriever-demo", dimension=1536)
```


```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，随之而来的是混乱",
        metadata={"year": 1993, "rating": 7.7, "genre": "动作"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥在梦中迷失，梦中又有梦...",
        metadata={"year": 2010, "director": "克里斯托弗·诺兰", "rating": 8.2},
    ),
    Document(
        page_content="一名心理学家/侦探在一系列梦中迷失，盗梦空间重新使用了这个概念",
        metadata={"year": 2006, "director": "今敏", "rating": 8.6},
    ),
    Document(
        page_content="一群普通身材的女性非常健康，一些男性为她们痴迷",
        metadata={"year": 2019, "director": "格蕾塔·葛韦格", "rating": 8.3},
    ),
    Document(
        page_content="玩具复活并乐在其中",
        metadata={"year": 1995, "genre": "动画"},
    ),
    Document(
        page_content="三名男子走进区域，三名男子走出区域",
        metadata={
            "year": 1979,
            "director": "安德烈·塔尔科夫斯基",
            "genre": "科幻",
            "rating": 9.9,
        },
    ),
]
vectorstore = DashVector.from_documents(
    docs, embeddings, collection_name="langchain-self-retriever-demo"
)
```

## 创建自查询检索器

现在我们可以实例化我们的检索器。为此，我们需要提前提供一些关于文档支持的元数据字段的信息以及文档内容的简短描述。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_community.llms import Tongyi

metadata_field_info = [
    AttributeInfo(
        name="genre",
        description="The genre of the movie",
        type="string or list[string]",
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
        name="rating", description="A 1-10 rating for the movie", type="float"
    ),
]
document_content_description = "Brief summary of a movie"
llm = Tongyi(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vectorstore, document_content_description, metadata_field_info, verbose=True
)
```

## 测试一下

现在我们可以尝试实际使用我们的检索器！


```python
# This example only specifies a relevant query
retriever.invoke("What are some movies about dinosaurs")
```
```output
query='dinosaurs' filter=None limit=None
```


```output
[Document(page_content='一群科学家复活了恐龙，随之而来的是混乱', metadata={'year': 1993, 'rating': 7.699999809265137, 'genre': 'action'}),
 Document(page_content='玩具复活并乐在其中', metadata={'year': 1995, 'genre': 'animated'}),
 Document(page_content='莱昂纳多·迪卡普里奥在梦中迷失，梦中又有梦，梦中再有梦……', metadata={'year': 2010, 'director': 'Christopher Nolan', 'rating': 8.199999809265137}),
 Document(page_content='一位心理学家/侦探在一系列梦中迷失，而《盗梦空间》重复了这个概念', metadata={'year': 2006, 'director': 'Satoshi Kon', 'rating': 8.600000381469727})]
```



```python
# This example only specifies a filter
retriever.invoke("I want to watch a movie rated higher than 8.5")
```
```output
query=' ' filter=Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='rating', value=8.5) limit=None
```


```output
[Document(page_content='三个人走进了区域，三个人走出了区域', metadata={'year': 1979, 'director': 'Andrei Tarkovsky', 'rating': 9.899999618530273, 'genre': 'science fiction'}),
 Document(page_content='一位心理学家/侦探在一系列梦中迷失，而《盗梦空间》重复了这个概念', metadata={'year': 2006, 'director': 'Satoshi Kon', 'rating': 8.600000381469727})]
```



```python
# This example specifies a query and a filter
retriever.invoke("Has Greta Gerwig directed any movies about women")
```
```output
query='Greta Gerwig' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig') limit=None
```


```output
[Document(page_content='一群普通身材的女性非常善良，一些男性对她们心怀向往', metadata={'year': 2019, 'director': 'Greta Gerwig', 'rating': 8.300000190734863})]
```



```python
# This example specifies a composite filter
retriever.invoke("What's a highly rated (above 8.5) science fiction film?")
```
```output
query='science fiction' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='science fiction'), Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5)]) limit=None
```


```output
[Document(page_content='三个人走进了区域，三个人走出了区域', metadata={'year': 1979, 'director': 'Andrei Tarkovsky', 'rating': 9.899999618530273, 'genre': 'science fiction'})]
```

## 过滤 k

我们还可以使用自查询检索器来指定 `k`：要获取的文档数量。

我们可以通过将 `enable_limit=True` 传递给构造函数来实现这一点。

```python
retriever = SelfQueryRetriever.from_llm(
    llm,
    vectorstore,
    document_content_description,
    metadata_field_info,
    enable_limit=True,
    verbose=True,
)
```

```python
# 此示例仅指定了一个相关查询
retriever.invoke("what are two movies about dinosaurs")
```
```output
query='dinosaurs' filter=None limit=2
```

```output
[Document(page_content='A bunch of scientists bring back dinosaurs and mayhem breaks loose', metadata={'year': 1993, 'rating': 7.699999809265137, 'genre': 'action'}),
 Document(page_content='Toys come alive and have a blast doing so', metadata={'year': 1995, 'genre': 'animated'})]
```