---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llm_caching.ipynb
---

# 模型缓存

本笔记本介绍了如何使用不同的缓存来缓存单个 LLM 调用的结果。

首先，让我们安装一些依赖项


```python
%pip install -qU langchain-openai langchain-community

import os
from getpass import getpass

os.environ["OPENAI_API_KEY"] = getpass()
```


```python
from langchain.globals import set_llm_cache
from langchain_openai import OpenAI

# 为了让缓存变得更加明显，我们使用一个较慢且较旧的模型。
# 缓存也支持更新的聊天模型。
llm = OpenAI(model="gpt-3.5-turbo-instruct", n=2, best_of=2)
```

## `In Memory` Cache


```python
from langchain_community.cache import InMemoryCache

set_llm_cache(InMemoryCache())
```


```python
%%time
# 第一次调用时，尚未缓存，因此应该需要更长的时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 7.57 ms, sys: 8.22 ms, total: 15.8 ms
Wall time: 649 ms
```


```output
"\n\nWhy couldn't the bicycle stand up by itself? Because it was two-tired!"
```



```python
%%time
# 第二次调用时，已经缓存，因此速度更快
llm.invoke("Tell me a joke")
```
```output
CPU times: user 551 µs, sys: 221 µs, total: 772 µs
Wall time: 1.23 ms
```


```output
"\n\nWhy couldn't the bicycle stand up by itself? Because it was two-tired!"
```

## `SQLite` 缓存


```python
!rm .langchain.db
```


```python
# 我们可以使用 SQLite 缓存做同样的事情
from langchain_community.cache import SQLiteCache

set_llm_cache(SQLiteCache(database_path=".langchain.db"))
```


```python
%%time
# 第一次，它还不在缓存中，所以会花更长的时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 12.6 ms, sys: 3.51 ms, total: 16.1 ms
Wall time: 486 ms
```


```output
"\n\n为什么自行车无法独自站立？因为它太累了！"
```



```python
%%time
# 第二次它已经在缓存中，所以会更快
llm.invoke("Tell me a joke")
```
```output
CPU times: user 52.6 ms, sys: 57.7 ms, total: 110 ms
Wall time: 113 ms
```


```output
"\n\n为什么自行车无法独自站立？因为它太累了！"
```

## `Upstash Redis` 缓存

### 标准缓存
使用 [Upstash Redis](https://upstash.com) 通过无服务器 HTTP API 缓存提示和响应。

```python
%pip install -qU upstash_redis
```

```python
import langchain
from langchain_community.cache import UpstashRedisCache
from upstash_redis import Redis

URL = "<UPSTASH_REDIS_REST_URL>"
TOKEN = "<UPSTASH_REDIS_REST_TOKEN>"

langchain.llm_cache = UpstashRedisCache(redis_=Redis(url=URL, token=TOKEN))
```

```python
%%time
# 第一次调用时，缓存中尚不存在，因此需要更长时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 7.56 ms, sys: 2.98 ms, total: 10.5 ms
Wall time: 1.14 s
```

```output
'\n\nWhy did the chicken cross the road?\n\nTo get to the other side!'
```

```python
%%time
# 第二次调用时，缓存中已存在，因此速度更快
llm.invoke("Tell me a joke")
```
```output
CPU times: user 2.78 ms, sys: 1.95 ms, total: 4.73 ms
Wall time: 82.9 ms
```

```output
'\n\nWhy did the chicken cross the road?\n\nTo get to the other side!'
```

## `Redis` 缓存

### 标准缓存
使用 [Redis](/docs/integrations/providers/redis) 来缓存提示和响应。


```python
%pip install -qU redis
```


```python
# 我们可以使用 Redis 缓存做同样的事情
# （在运行此示例之前，请确保您的本地 Redis 实例已启动）
from langchain_community.cache import RedisCache
from redis import Redis

set_llm_cache(RedisCache(redis_=Redis()))
```


```python
%%time
# 第一次，它还不在缓存中，因此应该需要更长时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 6.88 ms, sys: 8.75 ms, total: 15.6 ms
Wall time: 1.04 s
```


```output
'\n\n为什么鸡要过马路？\n\n为了到达另一边！'
```



```python
%%time
# 第二次它在缓存中，因此速度更快
llm.invoke("Tell me a joke")
```
```output
CPU times: user 1.59 ms, sys: 610 µs, total: 2.2 ms
Wall time: 5.58 ms
```


```output
'\n\n为什么鸡要过马路？\n\n为了到达另一边！'
```

### 语义缓存
使用 [Redis](/docs/integrations/providers/redis) 来缓存提示和响应，并根据语义相似性评估命中率。

```python
%pip install -qU redis
```

```python
from langchain_community.cache import RedisSemanticCache
from langchain_openai import OpenAIEmbeddings

set_llm_cache(
    RedisSemanticCache(redis_url="redis://localhost:6379", embedding=OpenAIEmbeddings())
)
```

```python
%%time
# 第一次，它还不在缓存中，所以应该需要更长时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 351 ms, sys: 156 ms, total: 507 ms
Wall time: 3.37 s
```

```output
"\n\nWhy don't scientists trust atoms?\nBecause they make up everything."
```

```python
%%time
# 第二次，虽然不是直接命中，但问题在语义上与原始问题相似，
# 因此使用缓存的结果！
llm.invoke("Tell me one joke")
```
```output
CPU times: user 6.25 ms, sys: 2.72 ms, total: 8.97 ms
Wall time: 262 ms
```

```output
"\n\nWhy don't scientists trust atoms?\nBecause they make up everything."
```

## `GPTCache`

我们可以使用 [GPTCache](https://github.com/zilliztech/GPTCache) 进行精确匹配缓存或基于语义相似性缓存结果

首先让我们从一个精确匹配的例子开始


```python
%pip install -qU gptcache
```


```python
import hashlib

from gptcache import Cache
from gptcache.manager.factory import manager_factory
from gptcache.processor.pre import get_prompt
from langchain_community.cache import GPTCache


def get_hashed_name(name):
    return hashlib.sha256(name.encode()).hexdigest()


def init_gptcache(cache_obj: Cache, llm: str):
    hashed_llm = get_hashed_name(llm)
    cache_obj.init(
        pre_embedding_func=get_prompt,
        data_manager=manager_factory(manager="map", data_dir=f"map_cache_{hashed_llm}"),
    )


set_llm_cache(GPTCache(init_gptcache))
```


```python
%%time
# 第一次，它还不在缓存中，所以应该花更长的时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 21.5 ms, sys: 21.3 ms, total: 42.8 ms
Wall time: 6.2 s
```


```output
'\n\n为什么鸡要过马路？\n\n为了到达另一边！'
```



```python
%%time
# 第二次它在缓存中，所以速度更快
llm.invoke("Tell me a joke")
```
```output
CPU times: user 571 µs, sys: 43 µs, total: 614 µs
Wall time: 635 µs
```


```output
'\n\n为什么鸡要过马路？\n\n为了到达另一边！'
```


现在让我们展示一个相似性缓存的例子


```python
import hashlib

from gptcache import Cache
from gptcache.adapter.api import init_similar_cache
from langchain_community.cache import GPTCache


def get_hashed_name(name):
    return hashlib.sha256(name.encode()).hexdigest()


def init_gptcache(cache_obj: Cache, llm: str):
    hashed_llm = get_hashed_name(llm)
    init_similar_cache(cache_obj=cache_obj, data_dir=f"similar_cache_{hashed_llm}")


set_llm_cache(GPTCache(init_gptcache))
```


```python
%%time
# 第一次，它还不在缓存中，所以应该花更长的时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 1.42 s, sys: 279 ms, total: 1.7 s
Wall time: 8.44 s
```


```output
'\n\n为什么鸡要过马路？\n\n为了到达另一边。'
```



```python
%%time
# 这是一个精确匹配，所以它在缓存中找到了
llm.invoke("Tell me a joke")
```
```output
CPU times: user 866 ms, sys: 20 ms, total: 886 ms
Wall time: 226 ms
```


```output
'\n\n为什么鸡要过马路？\n\n为了到达另一边。'
```



```python
%%time
# 这不是一个精确匹配，但在语义上在距离之内，所以它命中！
llm.invoke("Tell me joke")
```
```output
CPU times: user 853 ms, sys: 14.8 ms, total: 868 ms
Wall time: 224 ms
```


```output
'\n\n为什么鸡要过马路？\n\n为了到达另一边。'
```

## `MongoDB Atlas` 缓存

[MongoDB Atlas](https://www.mongodb.com/docs/atlas/) 是一个完全托管的云数据库，支持 AWS、Azure 和 GCP。它对 MongoDB 文档数据原生支持向量搜索。
使用 [MongoDB Atlas 向量搜索](/docs/integrations/providers/mongodb_atlas) 进行语义缓存提示和响应。

### `MongoDBCache`
一个用于在MongoDB中存储简单缓存的抽象。这不使用语义缓存，也不需要在生成之前在集合上创建索引。

要导入此缓存，首先安装所需的依赖项：

```bash
%pip install -qU langchain-mongodb
```

```python
from langchain_mongodb.cache import MongoDBCache
```


要将此缓存与您的LLM一起使用：
```python
from langchain_core.globals import set_llm_cache

# 使用任何嵌入提供者...
from tests.integration_tests.vectorstores.fake_embeddings import FakeEmbeddings

mongodb_atlas_uri = "<YOUR_CONNECTION_STRING>"
COLLECTION_NAME="<YOUR_CACHE_COLLECTION_NAME>"
DATABASE_NAME="<YOUR_DATABASE_NAME>"

set_llm_cache(MongoDBCache(
    connection_string=mongodb_atlas_uri,
    collection_name=COLLECTION_NAME,
    database_name=DATABASE_NAME,
))
```

### `MongoDBAtlasSemanticCache`
语义缓存允许用户根据用户输入与先前缓存结果之间的语义相似性检索缓存提示。在底层，它将MongoDBAtlas作为缓存和向量存储进行结合。
MongoDBAtlasSemanticCache继承自`MongoDBAtlasVectorSearch`，并需要定义一个Atlas向量搜索索引才能工作。请查看[使用示例](/docs/integrations/vectorstores/mongodb_atlas)以了解如何设置索引。

要导入此缓存：
```python
from langchain_mongodb.cache import MongoDBAtlasSemanticCache
```

要将此缓存与您的LLM一起使用：
```python
from langchain_core.globals import set_llm_cache

# 使用任何嵌入提供者...
from tests.integration_tests.vectorstores.fake_embeddings import FakeEmbeddings

mongodb_atlas_uri = "<YOUR_CONNECTION_STRING>"
COLLECTION_NAME="<YOUR_CACHE_COLLECTION_NAME>"
DATABASE_NAME="<YOUR_DATABASE_NAME>"

set_llm_cache(MongoDBAtlasSemanticCache(
    embedding=FakeEmbeddings(),
    connection_string=mongodb_atlas_uri,
    collection_name=COLLECTION_NAME,
    database_name=DATABASE_NAME,
))
```

要查找有关使用MongoDBSemanticCache的更多资源，请访问[这里](https://www.mongodb.com/blog/post/introducing-semantic-caching-dedicated-mongodb-lang-chain-package-gen-ai-apps)

## `Momento` 缓存
使用 [Momento](/docs/integrations/providers/momento) 来缓存提示和响应。

使用前需要安装 momento，取消下面的注释以进行安装：


```python
%pip install -qU momento
```

您需要获取一个 Momento 认证令牌才能使用此类。可以将其作为命名参数 `auth_token` 传递给 `MomentoChatMessageHistory.from_client_params`，也可以直接传递给 momento.CacheClient，或者将其设置为环境变量 `MOMENTO_AUTH_TOKEN`。


```python
from datetime import timedelta

from langchain_community.cache import MomentoCache

cache_name = "langchain"
ttl = timedelta(days=1)
set_llm_cache(MomentoCache.from_client_params(cache_name, ttl))
```


```python
%%time
# 第一次，它还不在缓存中，因此应该需要更长时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 40.7 ms, sys: 16.5 ms, total: 57.2 ms
Wall time: 1.73 s
```


```output
'\n\nWhy did the chicken cross the road?\n\nTo get to the other side!'
```



```python
%%time
# 第二次，它已经在缓存中，因此速度更快
# 在与缓存位于同一地区运行时，延迟为个位数毫秒
llm.invoke("Tell me a joke")
```
```output
CPU times: user 3.16 ms, sys: 2.98 ms, total: 6.14 ms
Wall time: 57.9 ms
```


```output
'\n\nWhy did the chicken cross the road?\n\nTo get to the other side!'
```

## `SQLAlchemy` 缓存

您可以使用 `SQLAlchemyCache` 在任何由 `SQLAlchemy` 支持的 SQL 数据库中进行缓存。

```python
# from langchain.cache import SQLAlchemyCache
# from sqlalchemy import create_engine

# engine = create_engine("postgresql://postgres:postgres@localhost:5432/postgres")
# set_llm_cache(SQLAlchemyCache(engine))
```

### 自定义 SQLAlchemy 模式


```python
# You can define your own declarative SQLAlchemyCache child class to customize the schema used for caching. For example, to support high-speed fulltext prompt indexing with Postgres, use:

from langchain_community.cache import SQLAlchemyCache
from sqlalchemy import Column, Computed, Index, Integer, Sequence, String, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy_utils import TSVectorType

Base = declarative_base()


class FulltextLLMCache(Base):  # type: ignore
    """Postgres table for fulltext-indexed LLM Cache"""

    __tablename__ = "llm_cache_fulltext"
    id = Column(Integer, Sequence("cache_id"), primary_key=True)
    prompt = Column(String, nullable=False)
    llm = Column(String, nullable=False)
    idx = Column(Integer)
    response = Column(String)
    prompt_tsv = Column(
        TSVectorType(),
        Computed("to_tsvector('english', llm || ' ' || prompt)", persisted=True),
    )
    __table_args__ = (
        Index("idx_fulltext_prompt_tsv", prompt_tsv, postgresql_using="gin"),
    )


engine = create_engine("postgresql://postgres:postgres@localhost:5432/postgres")
set_llm_cache(SQLAlchemyCache(engine, FulltextLLMCache))
```

## `Cassandra` 缓存

> [Apache Cassandra®](https://cassandra.apache.org/) 是一个 NoSQL、行导向、高度可扩展和高度可用的数据库。从 5.0 版本开始，数据库支持 [向量搜索功能](https://cassandra.apache.org/doc/trunk/cassandra/vector-search/overview.html)。

您可以使用 Cassandra 来缓存 LLM 响应，选择精确匹配的 `CassandraCache` 或基于向量相似性的 `CassandraSemanticCache`。

让我们看看这两者的实际应用。接下来的单元格将指导您完成（少量）所需的设置，后续单元格展示了两个可用的缓存类。

### 所需依赖项


```python
%pip install -qU "cassio>=0.1.4"
```

### 连接到数据库

本页面显示的 Cassandra 缓存可与 Cassandra 及其他衍生数据库一起使用，例如 Astra DB，它使用 CQL（Cassandra 查询语言）协议。

> DataStax [Astra DB](https://docs.datastax.com/en/astra-serverless/docs/vector-search/quickstart.html) 是一个基于 Cassandra 的托管无服务器数据库，提供相同的接口和优势。

根据您是通过 CQL 连接到 Cassandra 集群还是 Astra DB，您在实例化缓存（通过初始化 CassIO 连接）时将提供不同的参数。

#### 连接到 Cassandra 集群

您首先需要创建一个 `cassandra.cluster.Session` 对象，如 [Cassandra 驱动程序文档](https://docs.datastax.com/en/developer/python-driver/latest/api/cassandra/cluster/#module-cassandra.cluster) 中所述。具体细节各不相同（例如网络设置和身份验证），但这可能类似于：

```python
from cassandra.cluster import Cluster

cluster = Cluster(["127.0.0.1"])
session = cluster.connect()
```

现在，您可以将会话和所需的键空间名称设置为全局 CassIO 参数：

```python
import cassio

CASSANDRA_KEYSPACE = input("CASSANDRA_KEYSPACE = ")

cassio.init(session=session, keyspace=CASSANDRA_KEYSPACE)
```
```output
CASSANDRA_KEYSPACE =  demo_keyspace
```
#### 通过 CQL 连接到 Astra DB

在这种情况下，您使用以下连接参数初始化 CassIO：

- 数据库 ID，例如 `01234567-89ab-cdef-0123-456789abcdef`
- 令牌，例如 `AstraCS:6gBhNmsk135....`（必须是“数据库管理员”令牌）
- 可选的键空间名称（如果省略，将使用数据库的默认名称）

```python
import getpass

ASTRA_DB_ID = input("ASTRA_DB_ID = ")
ASTRA_DB_APPLICATION_TOKEN = getpass.getpass("ASTRA_DB_APPLICATION_TOKEN = ")

desired_keyspace = input("ASTRA_DB_KEYSPACE (optional, can be left empty) = ")
if desired_keyspace:
    ASTRA_DB_KEYSPACE = desired_keyspace
else:
    ASTRA_DB_KEYSPACE = None
```
```output
ASTRA_DB_ID =  01234567-89ab-cdef-0123-456789abcdef
ASTRA_DB_APPLICATION_TOKEN =  ········
ASTRA_DB_KEYSPACE (optional, can be left empty) =  my_keyspace
```

```python
import cassio

cassio.init(
    database_id=ASTRA_DB_ID,
    token=ASTRA_DB_APPLICATION_TOKEN,
    keyspace=ASTRA_DB_KEYSPACE,
)
```

### Cassandra: 精确缓存

这将避免在提供的提示与之前遇到的完全相同时调用LLM：

```python
from langchain_community.cache import CassandraCache
from langchain_core.globals import set_llm_cache

set_llm_cache(CassandraCache())
```

```python
%%time

print(llm.invoke("Why is the Moon always showing the same side?"))
```
```output


The Moon is tidally locked with the Earth, which means that its rotation on its own axis is synchronized with its orbit around the Earth. This results in the Moon always showing the same side to the Earth. This is because the gravitational forces between the Earth and the Moon have caused the Moon's rotation to slow down over time, until it reached a point where it takes the same amount of time for the Moon to rotate on its axis as it does to orbit around the Earth. This phenomenon is common among satellites in close orbits around their parent planets and is known as tidal locking.
CPU times: user 92.5 ms, sys: 8.89 ms, total: 101 ms
Wall time: 1.98 s
```

```python
%%time

print(llm.invoke("Why is the Moon always showing the same side?"))
```
```output


The Moon is tidally locked with the Earth, which means that its rotation on its own axis is synchronized with its orbit around the Earth. This results in the Moon always showing the same side to the Earth. This is because the gravitational forces between the Earth and the Moon have caused the Moon's rotation to slow down over time, until it reached a point where it takes the same amount of time for the Moon to rotate on its axis as it does to orbit around the Earth. This phenomenon is common among satellites in close orbits around their parent planets and is known as tidal locking.
CPU times: user 5.51 ms, sys: 0 ns, total: 5.51 ms
Wall time: 5.78 ms
```

### Cassandra: 语义缓存

此缓存将进行语义相似性搜索，如果找到足够相似的缓存条目，则返回命中。为此，您需要提供一个您选择的 `Embeddings` 实例。

```python
from langchain_openai import OpenAIEmbeddings

embedding = OpenAIEmbeddings()
```

```python
from langchain_community.cache import CassandraSemanticCache
from langchain_core.globals import set_llm_cache

set_llm_cache(
    CassandraSemanticCache(
        embedding=embedding,
        table_name="my_semantic_cache",
    )
)
```

```python
%%time

print(llm.invoke("为什么月球总是显示同一面？"))
```
```output


月球总是显示同一面是因为一种叫做同步旋转的现象。这意味着月球以与其绕地球公转相同的速度自转，公转大约需要27.3天。这导致月球的同一面始终朝向地球。这是由于地球和月球之间的引力作用，导致月球的自转逐渐减慢并与其公转同步。这在我们太阳系的许多卫星中都是一种常见现象。
CPU times: user 49.5 ms, sys: 7.38 ms, total: 56.9 ms
Wall time: 2.55 s
```

```python
%%time

print(llm.invoke("为什么我们总是看到月球的一面？"))
```
```output


月球总是显示同一面是因为一种叫做同步旋转的现象。这意味着月球以与其绕地球公转相同的速度自转，公转大约需要27.3天。这导致月球的同一面始终朝向地球。这是由于地球和月球之间的引力作用，导致月球的自转逐渐减慢并与其公转同步。这在我们太阳系的许多卫星中都是一种常见现象。
CPU times: user 21.2 ms, sys: 3.38 ms, total: 24.6 ms
Wall time: 532 ms
```
#### 版权声明

>Apache Cassandra、Cassandra 和 Apache 是 [Apache 软件基金会](http://www.apache.org/) 在美国和/或其他国家的注册商标或商标。

## `Astra DB` 缓存

您可以轻松地将 [Astra DB](https://docs.datastax.com/en/astra/home/astra.html) 用作 LLM 缓存，可以选择“精确”或“基于语义”的缓存。

确保您有一个正在运行的数据库（必须是支持向量的数据库才能使用语义缓存），并在您的 Astra 控制面板上获取所需的凭据：

- API 端点看起来像 `https://01234567-89ab-cdef-0123-456789abcdef-us-east1.apps.astra.datastax.com`
- 令牌看起来像 `AstraCS:6gBhNmsk135....`


```python
%pip install -qU langchain_astradb

import getpass

ASTRA_DB_API_ENDPOINT = input("ASTRA_DB_API_ENDPOINT = ")
ASTRA_DB_APPLICATION_TOKEN = getpass.getpass("ASTRA_DB_APPLICATION_TOKEN = ")
```
```output
ASTRA_DB_API_ENDPOINT =  https://01234567-89ab-cdef-0123-456789abcdef-us-east1.apps.astra.datastax.com
ASTRA_DB_APPLICATION_TOKEN =  ········
```

### Astra DB 精确 LLM 缓存

这将避免在提供的提示与已遇到的提示_完全_相同时调用 LLM：

```python
from langchain.globals import set_llm_cache
from langchain_astradb import AstraDBCache

set_llm_cache(
    AstraDBCache(
        api_endpoint=ASTRA_DB_API_ENDPOINT,
        token=ASTRA_DB_APPLICATION_TOKEN,
    )
)
```

```python
%%time

print(llm.invoke("Is a true fakery the same as a fake truth?"))
```
```output


There is no definitive answer to this question as it depends on the interpretation of the terms "true fakery" and "fake truth". However, one possible interpretation is that a true fakery is a counterfeit or imitation that is intended to deceive, whereas a fake truth is a false statement that is presented as if it were true.
CPU times: user 70.8 ms, sys: 4.13 ms, total: 74.9 ms
Wall time: 2.06 s
```

```python
%%time

print(llm.invoke("Is a true fakery the same as a fake truth?"))
```
```output


There is no definitive answer to this question as it depends on the interpretation of the terms "true fakery" and "fake truth". However, one possible interpretation is that a true fakery is a counterfeit or imitation that is intended to deceive, whereas a fake truth is a false statement that is presented as if it were true.
CPU times: user 15.1 ms, sys: 3.7 ms, total: 18.8 ms
Wall time: 531 ms
```

### Astra DB 语义缓存

该缓存将进行语义相似性搜索，如果找到足够相似的缓存条目，则返回命中。为此，您需要提供一个您选择的 `Embeddings` 实例。

```python
from langchain_openai import OpenAIEmbeddings

embedding = OpenAIEmbeddings()
```

```python
from langchain_astradb import AstraDBSemanticCache

set_llm_cache(
    AstraDBSemanticCache(
        api_endpoint=ASTRA_DB_API_ENDPOINT,
        token=ASTRA_DB_APPLICATION_TOKEN,
        embedding=embedding,
        collection_name="demo_semantic_cache",
    )
)
```

```python
%%time

print(llm.invoke("Are there truths that are false?"))
```
```output


There is no definitive answer to this question since it presupposes a great deal about the nature of truth itself, which is a matter of considerable philosophical debate. It is possible, however, to construct scenarios in which something could be considered true despite being false, such as if someone sincerely believes something to be true even though it is not.
CPU times: user 65.6 ms, sys: 15.3 ms, total: 80.9 ms
Wall time: 2.72 s
```

```python
%%time

print(llm.invoke("Is is possible that something false can be also true?"))
```
```output


There is no definitive answer to this question since it presupposes a great deal about the nature of truth itself, which is a matter of considerable philosophical debate. It is possible, however, to construct scenarios in which something could be considered true despite being false, such as if someone sincerely believes something to be true even though it is not.
CPU times: user 29.3 ms, sys: 6.21 ms, total: 35.5 ms
Wall time: 1.03 s
```

## Azure Cosmos DB 语义缓存

您可以使用这个集成的 [向量数据库](https://learn.microsoft.com/en-us/azure/cosmos-db/vector-database) 进行缓存。

```python
from langchain_community.cache import AzureCosmosDBSemanticCache
from langchain_community.vectorstores.azure_cosmos_db import (
    CosmosDBSimilarityType,
    CosmosDBVectorSearchType,
)
from langchain_openai import OpenAIEmbeddings

# Read more about Azure CosmosDB Mongo vCore vector search here https://learn.microsoft.com/en-us/azure/cosmos-db/mongodb/vcore/vector-search

NAMESPACE = "langchain_test_db.langchain_test_collection"
CONNECTION_STRING = (
    "Please provide your azure cosmos mongo vCore vector db connection string"
)

DB_NAME, COLLECTION_NAME = NAMESPACE.split(".")

# Default value for these params
num_lists = 3
dimensions = 1536
similarity_algorithm = CosmosDBSimilarityType.COS
kind = CosmosDBVectorSearchType.VECTOR_IVF
m = 16
ef_construction = 64
ef_search = 40
score_threshold = 0.9
application_name = "LANGCHAIN_CACHING_PYTHON"


set_llm_cache(
    AzureCosmosDBSemanticCache(
        cosmosdb_connection_string=CONNECTION_STRING,
        cosmosdb_client=None,
        embedding=OpenAIEmbeddings(),
        database_name=DB_NAME,
        collection_name=COLLECTION_NAME,
        num_lists=num_lists,
        similarity=similarity_algorithm,
        kind=kind,
        dimensions=dimensions,
        m=m,
        ef_construction=ef_construction,
        ef_search=ef_search,
        score_threshold=score_threshold,
        application_name=application_name,
    )
)
```


```python
%%time
# The first time, it is not yet in cache, so it should take longer
llm.invoke("Tell me a joke")
```
```output
CPU times: user 45.6 ms, sys: 19.7 ms, total: 65.3 ms
Wall time: 2.29 s
```


```output
'\n\nWhy was the math book sad? Because it had too many problems.'
```



```python
%%time
# The first time, it is not yet in cache, so it should take longer
llm.invoke("Tell me a joke")
```
```output
CPU times: user 9.61 ms, sys: 3.42 ms, total: 13 ms
Wall time: 474 ms
```


```output
'\n\nWhy was the math book sad? Because it had too many problems.'
```

## `Elasticsearch` 缓存
用于 LLM 的缓存层，使用 Elasticsearch。

首先安装与 Elasticsearch 的 LangChain 集成。

```python
%pip install -qU langchain-elasticsearch
```

使用类 `ElasticsearchCache`。

简单示例：

```python
from langchain.globals import set_llm_cache
from langchain_elasticsearch import ElasticsearchCache

set_llm_cache(
    ElasticsearchCache(
        es_url="http://localhost:9200",
        index_name="llm-chat-cache",
        metadata={"project": "my_chatgpt_project"},
    )
)
```

`index_name` 参数也可以接受别名。这允许使用 
[ILM: 管理索引生命周期](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html)
，建议考虑用于管理保留和控制缓存增长。

查看类文档字符串以获取所有参数。

### 索引生成的文本

缓存的数据默认不可搜索。
开发者可以自定义构建Elasticsearch文档，以便添加索引文本字段，
例如，将LLM生成的文本放置其中。

这可以通过子类化并重写方法来完成。
新的缓存类也可以应用于现有的缓存索引：


```python
import json
from typing import Any, Dict, List

from langchain.globals import set_llm_cache
from langchain_core.caches import RETURN_VAL_TYPE
from langchain_elasticsearch import ElasticsearchCache


class SearchableElasticsearchCache(ElasticsearchCache):
    @property
    def mapping(self) -> Dict[str, Any]:
        mapping = super().mapping
        mapping["mappings"]["properties"]["parsed_llm_output"] = {
            "type": "text",
            "analyzer": "english",
        }
        return mapping

    def build_document(
        self, prompt: str, llm_string: str, return_val: RETURN_VAL_TYPE
    ) -> Dict[str, Any]:
        body = super().build_document(prompt, llm_string, return_val)
        body["parsed_llm_output"] = self._parse_output(body["llm_output"])
        return body

    @staticmethod
    def _parse_output(data: List[str]) -> List[str]:
        return [
            json.loads(output)["kwargs"]["message"]["kwargs"]["content"]
            for output in data
        ]


set_llm_cache(
    SearchableElasticsearchCache(
        es_url="http://localhost:9200", index_name="llm-chat-cache"
    )
)
```

在重写映射和文档构建时，
请仅进行附加修改，保持基础映射不变。

## 可选缓存
您也可以选择为特定的 LLM 关闭缓存。在下面的例子中，尽管全局缓存已启用，但我们为特定的 LLM 关闭了缓存。

```python
llm = OpenAI(model="gpt-3.5-turbo-instruct", n=2, best_of=2, cache=False)
```

```python
%%time
llm.invoke("Tell me a joke")
```
```output
CPU times: user 5.8 ms, sys: 2.71 ms, total: 8.51 ms
Wall time: 745 ms
```

```output
'\n\n为什么鸡要过马路？\n\n为了到达另一边！'
```

```python
%%time
llm.invoke("Tell me a joke")
```
```output
CPU times: user 4.91 ms, sys: 2.64 ms, total: 7.55 ms
Wall time: 623 ms
```

```output
'\n\n两个家伙偷了一本日历。他们每人得了六个月。'
```

## 链中的可选缓存
您还可以为链中的特定节点关闭缓存。请注意，由于某些接口，通常先构建链，然后再编辑 LLM 更加方便。

作为示例，我们将加载一个摘要的 map-reduce 链。我们将为 map 步骤缓存结果，但不在 combine 步骤中冻结它。


```python
llm = OpenAI(model="gpt-3.5-turbo-instruct")
no_cache_llm = OpenAI(model="gpt-3.5-turbo-instruct", cache=False)
```


```python
from langchain_text_splitters import CharacterTextSplitter

text_splitter = CharacterTextSplitter()
```


```python
with open("../how_to/state_of_the_union.txt") as f:
    state_of_the_union = f.read()
texts = text_splitter.split_text(state_of_the_union)
```


```python
from langchain_core.documents import Document

docs = [Document(page_content=t) for t in texts[:3]]
from langchain.chains.summarize import load_summarize_chain
```


```python
chain = load_summarize_chain(llm, chain_type="map_reduce", reduce_llm=no_cache_llm)
```


```python
%%time
chain.invoke(docs)
```
```output
CPU times: user 176 ms, sys: 23.2 ms, total: 199 ms
Wall time: 4.42 s
```



当我们再次运行时，我们会看到它运行得快得多，但最终答案不同。这是由于在 map 步骤中缓存，但在 reduce 步骤中没有缓存。


```python
%%time
chain.invoke(docs)
```
```output
CPU times: user 7 ms, sys: 1.94 ms, total: 8.94 ms
Wall time: 1.06 s
```




```python
!rm .langchain.db sqlite.db
```
```output
rm: sqlite.db: No such file or directory
```

## OpenSearch 语义缓存
使用 [OpenSearch](https://python.langchain.com/docs/integrations/vectorstores/opensearch/) 作为语义缓存，以缓存提示和响应，并基于语义相似性评估命中。

```python
from langchain_community.cache import OpenSearchSemanticCache
from langchain_openai import OpenAIEmbeddings

set_llm_cache(
    OpenSearchSemanticCache(
        opensearch_url="http://localhost:9200", embedding=OpenAIEmbeddings()
    )
)
```

```python
%%time
# 第一次，它还不在缓存中，所以应该花更长的时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 39.4 ms, sys: 11.8 ms, total: 51.2 ms
Wall time: 1.55 s
```

```output
"\n\nWhy don't scientists trust atoms?\n\nBecause they make up everything."
```

```python
%%time
# 第二次，虽然不是直接命中，但问题在语义上与原始问题相似，
# 所以它使用缓存结果！
llm.invoke("Tell me one joke")
```
```output
CPU times: user 4.66 ms, sys: 1.1 ms, total: 5.76 ms
Wall time: 113 ms
```

```output
"\n\nWhy don't scientists trust atoms?\n\nBecause they make up everything."
```

## SingleStoreDB 语义缓存
您可以使用 [SingleStoreDB](https://python.langchain.com/docs/integrations/vectorstores/singlestoredb/) 作为语义缓存来缓存提示和响应。


```python
from langchain_community.cache import SingleStoreDBSemanticCache
from langchain_openai import OpenAIEmbeddings

set_llm_cache(
    SingleStoreDBSemanticCache(
        embedding=OpenAIEmbeddings(),
        host="root:pass@localhost:3306/db",
    )
)
```

## Couchbase 缓存

使用 [Couchbase](https://couchbase.com/) 作为提示和响应的缓存。

### Couchbase Cache

标准缓存会寻找用户提示的精确匹配。

```python
%pip install -qU langchain_couchbase couchbase
```

```python
# 创建 couchbase 连接对象
from datetime import timedelta

from couchbase.auth import PasswordAuthenticator
from couchbase.cluster import Cluster
from couchbase.options import ClusterOptions
from langchain_couchbase.cache import CouchbaseCache
from langchain_openai import ChatOpenAI

COUCHBASE_CONNECTION_STRING = (
    "couchbase://localhost"  # 或者 "couchbases://localhost" 如果使用 TLS
)
DB_USERNAME = "Administrator"
DB_PASSWORD = "Password"

auth = PasswordAuthenticator(DB_USERNAME, DB_PASSWORD)
options = ClusterOptions(auth)
cluster = Cluster(COUCHBASE_CONNECTION_STRING, options)

# 等待集群准备好使用。
cluster.wait_until_ready(timedelta(seconds=5))
```

```python
# 指定存储缓存文档的桶、范围和集合
BUCKET_NAME = "langchain-testing"
SCOPE_NAME = "_default"
COLLECTION_NAME = "_default"

set_llm_cache(
    CouchbaseCache(
        cluster=cluster,
        bucket_name=BUCKET_NAME,
        scope_name=SCOPE_NAME,
        collection_name=COLLECTION_NAME,
    )
)
```

```python
%%time
# 第一次，它还不在缓存中，因此应该需要更长时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 22.2 ms, sys: 14 ms, total: 36.2 ms
Wall time: 938 ms
```

```output
"\n\nWhy couldn't the bicycle stand up by itself? Because it was two-tired!"
```

```python
%%time
# 第二次，它已经在缓存中，因此应该快得多
llm.invoke("Tell me a joke")
```
```output
CPU times: user 53 ms, sys: 29 ms, total: 82 ms
Wall time: 84.2 ms
```

```output
"\n\nWhy couldn't the bicycle stand up by itself? Because it was two-tired!"
```

### Couchbase 语义缓存
语义缓存允许用户根据用户输入与之前缓存输入之间的语义相似性来检索缓存的提示。在底层，它使用 Couchbase 作为缓存和向量存储。这需要定义一个合适的向量搜索索引才能工作。请查看使用示例以了解如何设置索引。

```python
# Create Couchbase connection object
from datetime import timedelta

from couchbase.auth import PasswordAuthenticator
from couchbase.cluster import Cluster
from couchbase.options import ClusterOptions
from langchain_couchbase.cache import CouchbaseSemanticCache
from langchain_openai import ChatOpenAI, OpenAIEmbeddings

COUCHBASE_CONNECTION_STRING = (
    "couchbase://localhost"  # or "couchbases://localhost" if using TLS
)
DB_USERNAME = "Administrator"
DB_PASSWORD = "Password"

auth = PasswordAuthenticator(DB_USERNAME, DB_PASSWORD)
options = ClusterOptions(auth)
cluster = Cluster(COUCHBASE_CONNECTION_STRING, options)

# Wait until the cluster is ready for use.
cluster.wait_until_ready(timedelta(seconds=5))
```

注意：
- 在使用语义缓存之前，需要定义语义缓存的搜索索引。
- 可选参数 `score_threshold` 可以用来调整语义搜索的结果。

### 如何将索引导入全文搜索服务？
 - [Couchbase Server](https://docs.couchbase.com/server/current/search/import-search-index.html)
     - 点击搜索 -> 添加索引 -> 导入
     - 在导入界面复制以下索引定义
     - 点击创建索引以创建索引。
 - [Couchbase Capella](https://docs.couchbase.com/cloud/search/import-search-index.html)
     - 将索引定义复制到新文件 `index.json`
     - 按照文档中的说明在 Capella 中导入该文件。
     - 点击创建索引以创建索引。

#### 向量搜索的示例索引。 
  ```
  {
    "type": "fulltext-index",
    "name": "langchain-testing._default.semantic-cache-index",
    "sourceType": "gocbcore",
    "sourceName": "langchain-testing",
    "planParams": {
      "maxPartitionsPerPIndex": 1024,
      "indexPartitions": 16
    },
    "params": {
      "doc_config": {
        "docid_prefix_delim": "",
        "docid_regexp": "",
        "mode": "scope.collection.type_field",
        "type_field": "type"
      },
      "mapping": {
        "analysis": {},
        "default_analyzer": "standard",
        "default_datetime_parser": "dateTimeOptional",
        "default_field": "_all",
        "default_mapping": {
          "dynamic": true,
          "enabled": false
        },
        "default_type": "_default",
        "docvalues_dynamic": false,
        "index_dynamic": true,
        "store_dynamic": true,
        "type_field": "_type",
        "types": {
          "_default.semantic-cache": {
            "dynamic": false,
            "enabled": true,
            "properties": {
              "embedding": {
                "dynamic": false,
                "enabled": true,
                "fields": [
                  {
                    "dims": 1536,
                    "index": true,
                    "name": "embedding",
                    "similarity": "dot_product",
                    "type": "vector",
                    "vector_index_optimized_for": "recall"
                  }
                ]
              },
              "metadata": {
                "dynamic": true,
                "enabled": true
              },
              "text": {
                "dynamic": false,
                "enabled": true,
                "fields": [
                  {
                    "index": true,
                    "name": "text",
                    "store": true,
                    "type": "text"
                  }
                ]
              }
            }
          }
        }
      },
      "store": {
        "indexType": "scorch",
        "segmentVersion": 16
      }
    },
    "sourceParams": {}
  }
  ```


```python
BUCKET_NAME = "langchain-testing"
SCOPE_NAME = "_default"
COLLECTION_NAME = "semantic-cache"
INDEX_NAME = "semantic-cache-index"
embeddings = OpenAIEmbeddings()

cache = CouchbaseSemanticCache(
    cluster=cluster,
    embedding=embeddings,
    bucket_name=BUCKET_NAME,
    scope_name=SCOPE_NAME,
    collection_name=COLLECTION_NAME,
    index_name=INDEX_NAME,
    score_threshold=0.8,
)

set_llm_cache(cache)
```


```python
%%time
# 第一次，它还不在缓存中，因此应该花费更长时间
print(llm.invoke("狗的平均寿命是多少？"))
```
```output


狗的平均寿命约为12年，但这可能因品种、体型和个体狗的整体健康状况而异。一些小型犬可能活得更久，而大型犬的寿命可能较短。适当的护理、饮食和锻炼也可以在延长狗的寿命方面发挥作用。
CPU times: user 826 ms, sys: 2.46 s, total: 3.28 s
Wall time: 2.87 s
```

```python
%%time
# 第二次，它在缓存中，因此应该快得多
print(llm.invoke("狗的预期寿命是多少？"))
```
```output


狗的平均寿命约为12年，但这可能因品种、体型和个体狗的整体健康状况而异。一些小型犬可能活得更久，而大型犬的寿命可能较短。适当的护理、饮食和锻炼也可以在延长狗的寿命方面发挥作用。
CPU times: user 9.82 ms, sys: 2.61 ms, total: 12.4 ms
Wall time: 311 ms
```

## 缓存类：摘要表

**Cache** 类是通过继承 [BaseCache](https://api.python.langchain.com/en/latest/caches/langchain_core.caches.BaseCache.html) 类来实现的。

此表列出了所有 21 个派生类及其 API 参考链接。

| 命名空间 🔻 | 类 |
|------------|---------|
| langchain_astradb.cache | [AstraDBCache](https://api.python.langchain.com/en/latest/cache/langchain_astradb.cache.AstraDBCache.html) |
| langchain_astradb.cache | [AstraDBSemanticCache](https://api.python.langchain.com/en/latest/cache/langchain_astradb.cache.AstraDBSemanticCache.html) |
| langchain_community.cache | [AstraDBCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.AstraDBCache.html) |
| langchain_community.cache | [AstraDBSemanticCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.AstraDBSemanticCache.html) |
| langchain_community.cache | [AzureCosmosDBSemanticCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.AzureCosmosDBSemanticCache.html) |
| langchain_community.cache | [CassandraCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.CassandraCache.html) |
| langchain_community.cache | [CassandraSemanticCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.CassandraSemanticCache.html) |
| langchain_community.cache | [GPTCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.GPTCache.html) |
| langchain_community.cache | [InMemoryCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.InMemoryCache.html) |
| langchain_community.cache | [MomentoCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.MomentoCache.html) |
| langchain_community.cache | [OpenSearchSemanticCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.OpenSearchSemanticCache.html) |
| langchain_community.cache | [RedisSemanticCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.RedisSemanticCache.html) |
| langchain_community.cache | [SingleStoreDBSemanticCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.SingleStoreDBSemanticCache.html) |
| langchain_community.cache | [SQLAlchemyCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.SQLAlchemyCache.html) |
| langchain_community.cache | [SQLAlchemyMd5Cache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.SQLAlchemyMd5Cache.html) |
| langchain_community.cache | [UpstashRedisCache](https://api.python.langchain.com/en/latest/cache/langchain_community.cache.UpstashRedisCache.html) |
| langchain_core.caches | [InMemoryCache](https://api.python.langchain.com/en/latest/caches/langchain_core.caches.InMemoryCache.html) |
| langchain_elasticsearch.cache | [ElasticsearchCache](https://api.python.langchain.com/en/latest/cache/langchain_elasticsearch.cache.ElasticsearchCache.html) |
| langchain_mongodb.cache | [MongoDBAtlasSemanticCache](https://api.python.langchain.com/en/latest/cache/langchain_mongodb.cache.MongoDBAtlasSemanticCache.html) |
| langchain_mongodb.cache | [MongoDBCache](https://api.python.langchain.com/en/latest/cache/langchain_mongodb.cache.MongoDBCache.html) |
| langchain_couchbase.cache | [CouchbaseCache](https://api.python.langchain.com/en/latest/cache/langchain_couchbase.cache.CouchbaseCache.html) |
| langchain_couchbase.cache | [CouchbaseSemanticCache](https://api.python.langchain.com/en/latest/cache/langchain_couchbase.cache.CouchbaseSemanticCache.html) |