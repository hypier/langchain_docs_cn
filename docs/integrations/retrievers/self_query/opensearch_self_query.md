---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/self_query/opensearch_self_query.ipynb
---

# OpenSearch

> [OpenSearch](https://opensearch.org/) 是一款可扩展、灵活且可扩展的开源软件套件，适用于搜索、分析和可观察性应用，许可证为 Apache 2.0。`OpenSearch` 是一个基于 `Apache Lucene` 的分布式搜索和分析引擎。

在本笔记本中，我们将演示使用 `OpenSearch` 向量存储的 `SelfQueryRetriever`。

## 创建 OpenSearch 向量存储

首先，我们需要创建一个 `OpenSearch` 向量存储，并用一些数据进行初始化。我们创建了一小组包含电影摘要的演示文档。

**注意：** 自查询检索器需要您安装 `lark`（`pip install lark`）。我们还需要 `opensearch-py` 包。

```python
%pip install --upgrade --quiet  lark opensearch-py
```

```python
import getpass
import os

from langchain_community.vectorstores import OpenSearchVectorSearch
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")

embeddings = OpenAIEmbeddings()
```
```output
OpenAI API Key: ········
```

```python
docs = [
    Document(
        page_content="一群科学家带回恐龙，混乱随之而来",
        metadata={"year": 1993, "rating": 7.7, "genre": "科幻"},
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥迷失在梦中，梦中又有梦 ...",
        metadata={"year": 2010, "director": "克里斯托弗·诺兰", "rating": 8.2},
    ),
    Document(
        page_content="一位心理学家/侦探迷失在一系列梦中，梦中又有梦，《盗梦空间》重用了这个想法",
        metadata={"year": 2006, "director": "今敏", "rating": 8.6},
    ),
    Document(
        page_content="一群普通身材的女性极其健康，某些男性对她们心生向往",
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
            "rating": 9.9,
            "director": "安德烈·塔尔科夫斯基",
            "genre": "科幻",
        },
    ),
]
vectorstore = OpenSearchVectorSearch.from_documents(
    docs,
    embeddings,
    index_name="opensearch-self-query-demo",
    opensearch_url="http://localhost:9200",
)
```

## 创建自查询检索器
现在我们可以实例化我们的检索器。为此，我们需要提前提供一些关于我们的文档支持的元数据字段的信息，以及文档内容的简短描述。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import OpenAI

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
llm = OpenAI(temperature=0)
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
query='dinosaur' filter=None limit=None
```


```output
[Document(page_content='一群科学家复活了恐龙，随后混乱随之而来', metadata={'year': 1993, 'rating': 7.7, 'genre': 'science fiction'}),
 Document(page_content='玩具复活并乐在其中', metadata={'year': 1995, 'genre': 'animated'}),
 Document(page_content='莱昂纳多·迪卡普里奥在梦中迷失，梦中又有梦，梦中又有梦……', metadata={'year': 2010, 'director': 'Christopher Nolan', 'rating': 8.2}),
 Document(page_content='三个男人走进区域，三个男人走出区域', metadata={'year': 1979, 'rating': 9.9, 'director': 'Andrei Tarkovsky', 'genre': 'science fiction'})]
```



```python
# This example only specifies a filter
retriever.invoke("I want to watch a movie rated higher than 8.5")
```
```output
query=' ' filter=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.5) limit=None
```


```output
[Document(page_content='三个男人走进区域，三个男人走出区域', metadata={'year': 1979, 'rating': 9.9, 'director': 'Andrei Tarkovsky', 'genre': 'science fiction'}),
 Document(page_content='一位心理学家/侦探在一系列梦中迷失，梦中又有梦，且《盗梦空间》重用了这个概念', metadata={'year': 2006, 'director': 'Satoshi Kon', 'rating': 8.6})]
```



```python
# This example specifies a query and a filter
retriever.invoke("Has Greta Gerwig directed any movies about women")
```
```output
query='women' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig') limit=None
```


```output
[Document(page_content='一群正常身材的女性非常纯真，一些男性对她们心怀爱慕', metadata={'year': 2019, 'director': 'Greta Gerwig', 'rating': 8.3})]
```



```python
# This example specifies a composite filter
retriever.invoke("What's a highly rated (above 8.5) science fiction film?")
```
```output
query=' ' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='rating', value=8.5), Comparison(comparator=<Comparator.CONTAIN: 'contain'>, attribute='genre', value='science fiction')]) limit=None
```


```output
[Document(page_content='三个男人走进区域，三个男人走出区域', metadata={'year': 1979, 'rating': 9.9, 'director': 'Andrei Tarkovsky', 'genre': 'science fiction'})]
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
# 这个例子只指定了一个相关的查询
retriever.invoke("what are two movies about dinosaurs")
```
```output
query='dinosaur' filter=None limit=2
```

```output
[Document(page_content='A bunch of scientists bring back dinosaurs and mayhem breaks loose', metadata={'year': 1993, 'rating': 7.7, 'genre': 'science fiction'}),
 Document(page_content='Toys come alive and have a blast doing so', metadata={'year': 1995, 'genre': 'animated'})]
```

## 复杂查询的实践！
我们尝试了一些简单的查询，但更复杂的查询呢？让我们尝试一些利用 OpenSearch 全力的复杂查询。

```python
retriever.invoke(
    "what animated or comedy movies have been released in the last 30 years about animated toys?"
)
```
```output
query='animated toys' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Operation(operator=<Operator.OR: 'or'>, arguments=[Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='animated'), Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='genre', value='comedy')]), Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='year', value=1990)]) limit=None
```

```output
[Document(page_content='Toys come alive and have a blast doing so', metadata={'year': 1995, 'genre': 'animated'})]
```

```python
vectorstore.client.indices.delete(index="opensearch-self-query-demo")
```

```output
{'acknowledged': True}
```