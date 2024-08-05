---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/self_query/redis_self_query.ipynb
---

# Redis

>[Redis](https://redis.com) 是一个开源的键值存储，可以用作缓存、消息代理、数据库、向量数据库等。

在笔记本中，我们将演示围绕 `Redis` 向量存储的 `SelfQueryRetriever`。

## 创建 Redis 向量存储
首先，我们需要创建一个 Redis 向量存储，并用一些数据进行初始化。我们创建了一小组包含电影摘要的演示文档。

**注意：** 自查询检索器需要您安装 `lark`（`pip install lark`）以及特定于集成的其他要求。


```python
%pip install --upgrade --quiet  redis redisvl langchain-openai tiktoken lark
```

我们想使用 `OpenAIEmbeddings`，因此必须获取 OpenAI API 密钥。


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```


```python
from langchain_community.vectorstores import Redis
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
```


```python
docs = [
    Document(
        page_content="一群科学家复活了恐龙，随之而来的是混乱",
        metadata={
            "year": 1993,
            "rating": 7.7,
            "director": "斯蒂文·斯皮尔伯格",
            "genre": "科幻",
        },
    ),
    Document(
        page_content="莱昂纳多·迪卡普里奥在梦中迷失，梦中又是一个梦，梦中又是一个梦...",
        metadata={
            "year": 2010,
            "director": "克里斯托弗·诺兰",
            "genre": "科幻",
            "rating": 8.2,
        },
    ),
    Document(
        page_content="一名心理学家/侦探在一系列梦中迷失，梦中又是梦，而《盗梦空间》重复了这个概念",
        metadata={
            "year": 2006,
            "director": "今敏",
            "genre": "科幻",
            "rating": 8.6,
        },
    ),
    Document(
        page_content="一群普通身材的女性极其健康，一些男性对她们心生向往",
        metadata={
            "year": 2019,
            "director": "格蕾塔·葛韦格",
            "genre": "剧情",
            "rating": 8.3,
        },
    ),
    Document(
        page_content="玩具们活了过来，玩得不亦乐乎",
        metadata={
            "year": 1995,
            "director": "约翰·拉塞特",
            "genre": "动画",
            "rating": 9.1,
        },
    ),
    Document(
        page_content="三名男子走进禁区，三名男子走出禁区",
        metadata={
            "year": 1979,
            "rating": 9.9,
            "director": "安德烈·塔可夫斯基",
            "genre": "科幻",
        },
    ),
]
```


```python
index_schema = {
    "tag": [{"name": "genre"}],
    "text": [{"name": "director"}],
    "numeric": [{"name": "year"}, {"name": "rating"}],
}

vectorstore = Redis.from_documents(
    docs,
    embeddings,
    redis_url="redis://localhost:6379",
    index_name="movie_reviews",
    index_schema=index_schema,
)
```
```output
`index_schema` does not match generated metadata schema.
If you meant to manually override the schema, please ignore this message.
index_schema: {'tag': [{'name': 'genre'}], 'text': [{'name': 'director'}], 'numeric': [{'name': 'year'}, {'name': 'rating'}]}
generated_schema: {'text': [{'name': 'director'}, {'name': 'genre'}], 'numeric': [{'name': 'year'}, {'name': 'rating'}], 'tag': []}
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
```

```python
llm = OpenAI(temperature=0)
retriever = SelfQueryRetriever.from_llm(
    llm, vectorstore, document_content_description, metadata_field_info, verbose=True
)
```

## 测试一下
现在我们可以尝试实际使用我们的检索器！


```python
# 这个例子只指定了一个相关查询
retriever.invoke("What are some movies about dinosaurs")
```
```output
/Users/bagatur/langchain/libs/langchain/langchain/chains/llm.py:278: UserWarning: The predict_and_parse method is deprecated, instead pass an output parser directly to LLMChain.
  warnings.warn(
``````output
query='dinosaur' filter=None limit=None
```


```output
[Document(page_content='一群科学家复活了恐龙，混乱随之而来', metadata={'id': 'doc:movie_reviews:7b5481d753bc4135851b66fa61def7fb', 'director': 'Steven Spielberg', 'genre': 'science fiction', 'year': '1993', 'rating': '7.7'}),
 Document(page_content='玩具复活并尽情玩乐', metadata={'id': 'doc:movie_reviews:9e4e84daa0374941a6aa4274e9bbb607', 'director': 'John Lasseter', 'genre': 'animated', 'year': '1995', 'rating': '9.1'}),
 Document(page_content='三个人走进了区域，三个人走出了区域', metadata={'id': 'doc:movie_reviews:2cc66f38bfbd438eb3a045d90a1a4088', 'director': 'Andrei Tarkovsky', 'genre': 'science fiction', 'year': '1979', 'rating': '9.9'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦中，梦中又有梦，且《盗梦空间》借用了这个想法', metadata={'id': 'doc:movie_reviews:edf567b1d5334e02b2a4c692d853c80c', 'director': 'Satoshi Kon', 'genre': 'science fiction', 'year': '2006', 'rating': '8.6'})]
```



```python
# 这个例子只指定了一个过滤器
retriever.invoke("I want to watch a movie rated higher than 8.4")
```
```output
query=' ' filter=Comparison(comparator=<Comparator.GT: 'gt'>, attribute='rating', value=8.4) limit=None
```


```output
[Document(page_content='玩具复活并尽情玩乐', metadata={'id': 'doc:movie_reviews:9e4e84daa0374941a6aa4274e9bbb607', 'director': 'John Lasseter', 'genre': 'animated', 'year': '1995', 'rating': '9.1'}),
 Document(page_content='三个人走进了区域，三个人走出了区域', metadata={'id': 'doc:movie_reviews:2cc66f38bfbd438eb3a045d90a1a4088', 'director': 'Andrei Tarkovsky', 'genre': 'science fiction', 'year': '1979', 'rating': '9.9'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦中，梦中又有梦，且《盗梦空间》借用了这个想法', metadata={'id': 'doc:movie_reviews:edf567b1d5334e02b2a4c692d853c80c', 'director': 'Satoshi Kon', 'genre': 'science fiction', 'year': '2006', 'rating': '8.6'})]
```



```python
# 这个例子指定了一个查询和一个过滤器
retriever.invoke("Has Greta Gerwig directed any movies about women")
```
```output
query='women' filter=Comparison(comparator=<Comparator.EQ: 'eq'>, attribute='director', value='Greta Gerwig') limit=None
```


```output
[Document(page_content='一群正常身材的女性非常善良，一些男性渴望她们', metadata={'id': 'doc:movie_reviews:bb899807b93c442083fd45e75a4779d5', 'director': 'Greta Gerwig', 'genre': 'drama', 'year': '2019', 'rating': '8.3'})]
```



```python
# 这个例子指定了一个复合过滤器
retriever.invoke("What's a highly rated (above 8.5) science fiction film?")
```
```output
query=' ' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GTE: 'gte'>, attribute='rating', value=8.5), Comparison(comparator=<Comparator.CONTAIN: 'contain'>, attribute='genre', value='science fiction')]) limit=None
```


```output
[Document(page_content='三个人走进了区域，三个人走出了区域', metadata={'id': 'doc:movie_reviews:2cc66f38bfbd438eb3a045d90a1a4088', 'director': 'Andrei Tarkovsky', 'genre': 'science fiction', 'year': '1979', 'rating': '9.9'}),
 Document(page_content='一位心理学家/侦探迷失在一系列梦中，梦中又有梦，且《盗梦空间》借用了这个想法', metadata={'id': 'doc:movie_reviews:edf567b1d5334e02b2a4c692d853c80c', 'director': 'Satoshi Kon', 'genre': 'science fiction', 'year': '2006', 'rating': '8.6'})]
```



```python
# 这个例子指定了一个查询和复合过滤器
retriever.invoke(
    "What's a movie after 1990 but before 2005 that's all about toys, and preferably is animated"
)
```
```output
query='toys' filter=Operation(operator=<Operator.AND: 'and'>, arguments=[Comparison(comparator=<Comparator.GT: 'gt'>, attribute='year', value=1990), Comparison(comparator=<Comparator.LT: 'lt'>, attribute='year', value=2005), Comparison(comparator=<Comparator.CONTAIN: 'contain'>, attribute='genre', value='animated')]) limit=None
```


```output
[Document(page_content='玩具复活并尽情玩乐', metadata={'id': 'doc:movie_reviews:9e4e84daa0374941a6aa4274e9bbb607', 'director': 'John Lasseter', 'genre': 'animated', 'year': '1995', 'rating': '9.1'})]
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
# 此示例仅指定相关查询
retriever.invoke("what are two movies about dinosaurs")
```
```output
query='dinosaur' filter=None limit=2
```

```output
[Document(page_content='A bunch of scientists bring back dinosaurs and mayhem breaks loose', metadata={'id': 'doc:movie_reviews:7b5481d753bc4135851b66fa61def7fb', 'director': 'Steven Spielberg', 'genre': 'science fiction', 'year': '1993', 'rating': '7.7'}),
 Document(page_content='Toys come alive and have a blast doing so', metadata={'id': 'doc:movie_reviews:9e4e84daa0374941a6aa4274e9bbb607', 'director': 'John Lasseter', 'genre': 'animated', 'year': '1995', 'rating': '9.1'})]
```