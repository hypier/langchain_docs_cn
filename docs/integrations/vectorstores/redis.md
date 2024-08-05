---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/redis.ipynb
---

# Redis

>[Redis 向量数据库](https://redis.io/docs/get-started/vector-database/) 介绍及 langchain 集成指南。

## 什么是 Redis？

大多数来自网络服务背景的开发人员都熟悉 `Redis`。从本质上讲，`Redis` 是一个开源的键值存储，用作缓存、消息代理和数据库。开发人员选择 `Redis` 是因为它速度快，拥有庞大的客户端库生态系统，并且多年来已被主要企业部署。

除了这些传统用例外，`Redis` 还提供了额外的功能，如搜索和查询能力，使用户能够在 `Redis` 内创建二级索引结构。这使得 `Redis` 能够以缓存的速度作为向量数据库。

## Redis作为向量数据库

`Redis`使用压缩的倒排索引进行快速索引，同时占用较低的内存。它还支持多项高级功能，例如：

* 在Redis哈希和`JSON`中对多个字段进行索引
* 向量相似性搜索（使用`HNSW`（近似最近邻）或`FLAT`（精确最近邻））
* 向量范围搜索（例如，查找与查询向量在半径内的所有向量）
* 增量索引而不影响性能
* 文档排序（使用[tf-idf](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)，可选用户提供的权重）
* 字段加权
* 使用`AND`、`OR`和`NOT`运算符的复杂布尔查询
* 前缀匹配、模糊匹配和精确短语查询
* 支持[双元音匹配](https://redis.io/docs/stack/search/reference/phonetic_matching/)
* 自动补全建议（带有模糊前缀建议）
* 在[多种语言](https://redis.io/docs/stack/search/reference/stemming/)中基于词干的查询扩展（使用[Snowball](http://snowballstem.org/)）
* 支持中文分词和查询（使用[Friso](https://github.com/lionsoul2014/friso)）
* 数值过滤和范围
* 使用Redis地理空间索引进行地理空间搜索
* 强大的聚合引擎
* 支持所有`utf-8`编码的文本
* 检索完整文档、选定字段或仅文档ID
* 排序结果（例如，按创建日期）

## 客户端

由于 `Redis` 不仅仅是一个向量数据库，因此通常会有一些用例需要使用 `Redis` 客户端，而不仅仅是 `LangChain` 集成。您可以使用任何标准的 `Redis` 客户端库来运行搜索和查询命令，但使用一个封装了搜索和查询 API 的库会更简单。以下是一些示例，您可以在 [这里](https://redis.io/resources/clients/) 找到更多客户端库。

| 项目 | 语言 | 许可证 | 作者 | 星标 |
|----------|---------|--------|---------|-------|
| [jedis][jedis-url] | Java | MIT | [Redis][redis-url] | ![Stars][jedis-stars] |
| [redisvl][redisvl-url] | Python | MIT | [Redis][redis-url] | ![Stars][redisvl-stars] |
| [redis-py][redis-py-url] | Python | MIT | [Redis][redis-url] | ![Stars][redis-py-stars] |
| [node-redis][node-redis-url] | Node.js | MIT | [Redis][redis-url] | ![Stars][node-redis-stars] |
| [nredisstack][nredisstack-url] | .NET | MIT | [Redis][redis-url] | ![Stars][nredisstack-stars] |

[redis-url]: https://redis.com

[redisvl-url]: https://github.com/RedisVentures/redisvl
[redisvl-stars]: https://img.shields.io/github/stars/RedisVentures/redisvl.svg?style=social&amp;label=Star&amp;maxAge=2592000
[redisvl-package]: https://pypi.python.org/pypi/redisvl

[redis-py-url]: https://github.com/redis/redis-py
[redis-py-stars]: https://img.shields.io/github/stars/redis/redis-py.svg?style=social&amp;label=Star&amp;maxAge=2592000
[redis-py-package]: https://pypi.python.org/pypi/redis

[jedis-url]: https://github.com/redis/jedis
[jedis-stars]: https://img.shields.io/github/stars/redis/jedis.svg?style=social&amp;label=Star&amp;maxAge=2592000
[Jedis-package]: https://search.maven.org/artifact/redis.clients/jedis

[nredisstack-url]: https://github.com/redis/nredisstack
[nredisstack-stars]: https://img.shields.io/github/stars/redis/nredisstack.svg?style=social&amp;label=Star&amp;maxAge=2592000
[nredisstack-package]: https://www.nuget.org/packages/nredisstack/

[node-redis-url]: https://github.com/redis/node-redis
[node-redis-stars]: https://img.shields.io/github/stars/redis/node-redis.svg?style=social&amp;label=Star&amp;maxAge=2592000
[node-redis-package]: https://www.npmjs.com/package/redis

[redis-om-python-url]: https://github.com/redis/redis-om-python
[redis-om-python-author]: https://redis.com
[redis-om-python-stars]: https://img.shields.io/github/stars/redis/redis-om-python.svg?style=social&amp;label=Star&amp;maxAge=2592000

[redisearch-go-url]: https://github.com/RediSearch/redisearch-go
[redisearch-go-author]: https://redis.com
[redisearch-go-stars]: https://img.shields.io/github/stars/RediSearch/redisearch-go.svg?style=social&amp;label=Star&amp;maxAge=2592000

[redisearch-api-rs-url]: https://github.com/RediSearch/redisearch-api-rs
[redisearch-api-rs-author]: https://redis.com
[redisearch-api-rs-stars]: https://img.shields.io/github/stars/RediSearch/redisearch-api-rs.svg?style=social&amp;label=Star&amp;maxAge=2592000

## 部署选项

有许多方法可以将 Redis 与 RediSearch 部署在一起。开始的最简单方法是使用 Docker，但还有许多潜在的部署选项，例如

- [Redis Cloud](https://redis.com/redis-enterprise-cloud/overview/)
- [Docker (Redis Stack)](https://hub.docker.com/r/redis/redis-stack)
- 云市场： [AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-e6y7ork67pjwg?sr=0-2&ref_=beagle&applicationId=AWSMPContessa)， [Google Marketplace](https://console.cloud.google.com/marketplace/details/redislabs-public/redis-enterprise?pli=1)，或 [Azure Marketplace](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/garantiadata.redis_enterprise_1sp_public_preview?tab=Overview)
- 本地部署： [Redis Enterprise Software](https://redis.com/redis-enterprise-software/overview/)
- Kubernetes： [Redis Enterprise Software on Kubernetes](https://docs.redis.com/latest/kubernetes/)

## 其他示例

许多示例可以在 [Redis AI 团队的 GitHub](https://github.com/RedisVentures/) 中找到

- [Awesome Redis AI Resources](https://github.com/RedisVentures/redis-ai-resources) - 使用 Redis 进行 AI 工作负载的示例列表
- [Azure OpenAI Embeddings Q&A](https://github.com/ruoccofabrizio/azure-open-ai-embeddings-qna) - OpenAI 和 Redis 作为 Azure 上的问答服务。
- [ArXiv Paper Search](https://github.com/RedisVentures/redis-arXiv-search) - 对 arXiv 学术论文的语义搜索
- [Vector Search on Azure](https://learn.microsoft.com/azure/azure-cache-for-redis/cache-tutorial-vector-similarity) - 使用 Azure Cache for Redis 和 Azure OpenAI 在 Azure 上进行向量搜索

## 更多资源

有关如何将 Redis 用作向量数据库的更多信息，请查看以下资源：

- [RedisVL 文档](https://redisvl.com) - Redis 向量库客户端的文档
- [Redis 向量相似度文档](https://redis.io/docs/stack/search/reference/vectors/) - Redis 官方的向量搜索文档。
- [Redis-py 搜索文档](https://redis.readthedocs.io/en/latest/redismodules.html#redisearch-commands) - redis-py 客户端库的文档
- [向量相似度搜索：从基础到生产](https://mlops.community/vector-similarity-search-from-basics-to-production/) - 关于 VSS 和 Redis 作为 VectorDB 的入门博客文章。

## 设置

### 安装 Redis Python 客户端

`Redis-py` 是 Redis 官方支持的客户端。最近发布了专为向量数据库用例设计的 `RedisVL` 客户端。两者都可以通过 pip 安装。

```python
%pip install --upgrade --quiet  redis redisvl langchain-openai tiktoken
```

我们想使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
```

### 本地部署 Redis

要在本地部署 Redis，请运行：

```console
docker run -d -p 6379:6379 -p 8001:8001 redis/redis-stack:latest
```
如果一切正常，您应该可以在 `http://localhost:8001` 看到一个漂亮的 Redis 用户界面。请参阅上面的 [Deployment options](#deployment-options) 部分以获取其他部署方式。

### Redis 连接 URL 方案

有效的 Redis URL 方案有：
1. `redis://`  - 连接到 Redis 独立实例，未加密
2. `rediss://` - 连接到 Redis 独立实例，使用 TLS 加密
3. `redis+sentinel://`  - 通过 Redis Sentinel 连接到 Redis 服务器，未加密
4. `rediss+sentinel://` - 通过 Redis Sentinel 连接到 Redis 服务器，两个连接均使用 TLS 加密

有关其他连接参数的更多信息，请参见 [redis-py 文档](https://redis-py.readthedocs.io/en/stable/connections.html)。

```python
# connection to redis standalone at localhost, db 0, no password
redis_url = "redis://localhost:6379"
# connection to host "redis" port 7379 with db 2 and password "secret" (old style authentication scheme without username / pre 6.x)
redis_url = "redis://:secret@redis:7379/2"
# connection to host redis on default port with user "joe", pass "secret" using redis version 6+ ACLs
redis_url = "redis://joe:secret@redis/0"

# connection to sentinel at localhost with default group mymaster and db 0, no password
redis_url = "redis+sentinel://localhost:26379"
# connection to sentinel at host redis with default port 26379 and user "joe" with password "secret" with default group mymaster and db 0
redis_url = "redis+sentinel://joe:secret@redis"
# connection to sentinel, no auth with sentinel monitoring group "zone-1" and database 2
redis_url = "redis+sentinel://redis:26379/zone-1/2"

# connection to redis standalone at localhost, db 0, no password but with TLS support
redis_url = "rediss://localhost:6379"
# connection to redis sentinel at localhost and default port, db 0, no password
# but with TLS support for booth Sentinel and Redis server
redis_url = "rediss+sentinel://localhost"
```

### 示例数据

首先，我们将描述一些示例数据，以便演示 Redis 向量存储的各种属性。

```python
metadata = [
    {
        "user": "john",
        "age": 18,
        "job": "engineer",
        "credit_score": "high",
    },
    {
        "user": "derrick",
        "age": 45,
        "job": "doctor",
        "credit_score": "low",
    },
    {
        "user": "nancy",
        "age": 94,
        "job": "doctor",
        "credit_score": "high",
    },
    {
        "user": "tyler",
        "age": 100,
        "job": "engineer",
        "credit_score": "high",
    },
    {
        "user": "joe",
        "age": 35,
        "job": "dentist",
        "credit_score": "medium",
    },
]
texts = ["foo", "foo", "foo", "bar", "bar"]
```

### 创建 Redis 向量存储

Redis VectorStore 实例可以通过多种方式初始化。有多种类方法可以用来初始化 Redis VectorStore 实例。

- ``Redis.__init__`` - 直接初始化
- ``Redis.from_documents`` - 从一组 ``Langchain.docstore.Document`` 对象初始化
- ``Redis.from_texts`` - 从一组文本初始化（可选附带元数据）
- ``Redis.from_texts_return_keys`` - 从一组文本初始化（可选附带元数据）并返回键
- ``Redis.from_existing_index`` - 从现有的 Redis 索引初始化

下面我们将使用 ``Redis.from_texts`` 方法。


```python
from langchain_community.vectorstores.redis import Redis

rds = Redis.from_texts(
    texts,
    embeddings,
    metadatas=metadata,
    redis_url="redis://localhost:6379",
    index_name="users",
)
```


```python
rds.index_name
```



```output
'users'
```

## 检查创建的索引

一旦构建了``Redis`` VectorStore对象，如果索引尚不存在，则会在Redis中创建一个索引。可以使用``rvl``和``redis-cli``命令行工具检查该索引。如果您在上面安装了``redisvl``，可以使用``rvl``命令行工具来检查索引。

```python
# 假设您在本地运行Redis（使用 --host, --port, --password, --username 来更改此设置）
!rvl index listall
```
```output
[32m16:58:26[0m [34m[RedisVL][0m [1;30mINFO[0m   Indices:
[32m16:58:26[0m [34m[RedisVL][0m [1;30mINFO[0m   1. users
```
``Redis`` VectorStore实现将尝试为通过``from_texts``、``from_texts_return_keys``和``from_documents``方法传递的任何元数据生成索引模式（过滤字段）。这样，传递的任何元数据都将被索引到Redis搜索索引中，从而允许对这些字段进行过滤。

下面我们展示了从我们上面定义的元数据中创建的字段

```python
!rvl index info -i users
```
```output


Index Information:
╭──────────────┬────────────────┬───────────────┬─────────────────┬────────────╮
│ Index Name   │ Storage Type   │ Prefixes      │ Index Options   │   Indexing │
├──────────────┼────────────────┼───────────────┼─────────────────┼────────────┤
│ users        │ HASH           │ ['doc:users'] │ []              │          0 │
╰──────────────┴────────────────┴───────────────┴─────────────────┴────────────╯
Index Fields:
╭────────────────┬────────────────┬─────────┬────────────────┬────────────────╮
│ Name           │ Attribute      │ Type    │ Field Option   │   Option Value │
├────────────────┼────────────────┼─────────┼────────────────┼────────────────┤
│ user           │ user           │ TEXT    │ WEIGHT         │              1 │
│ job            │ job            │ TEXT    │ WEIGHT         │              1 │
│ credit_score   │ credit_score   │ TEXT    │ WEIGHT         │              1 │
│ content        │ content        │ TEXT    │ WEIGHT         │              1 │
│ age            │ age            │ NUMERIC │                │                │
│ content_vector │ content_vector │ VECTOR  │                │                │
╰────────────────┴────────────────┴─────────┴────────────────┴────────────────╯
```

```python
!rvl stats -i users
```
```output

Statistics:
╭─────────────────────────────┬─────────────╮
│ Stat Key                    │ Value       │
├─────────────────────────────┼─────────────┤
│ num_docs                    │ 5           │
│ num_terms                   │ 15          │
│ max_doc_id                  │ 5           │
│ num_records                 │ 33          │
│ percent_indexed             │ 1           │
│ hash_indexing_failures      │ 0           │
│ number_of_uses              │ 4           │
│ bytes_per_record_avg        │ 4.60606     │
│ doc_table_size_mb           │ 0.000524521 │
│ inverted_sz_mb              │ 0.000144958 │
│ key_table_size_mb           │ 0.000193596 │
│ offset_bits_per_record_avg  │ 8           │
│ offset_vectors_sz_mb        │ 2.19345e-05 │
│ offsets_per_term_avg        │ 0.69697     │
│ records_per_doc_avg         │ 6.6         │
│ sortable_values_size_mb     │ 0           │
│ total_indexing_time         │ 0.32        │
│ total_inverted_index_blocks │ 16          │
│ vector_index_sz_mb          │ 6.0126      │
╰─────────────────────────────┴─────────────╯
```
需要注意的是，我们并未指定元数据中的``user``、``job``、``credit_score``和``age``应作为索引中的字段，这是因为``Redis`` VectorStore对象会自动根据传递的元数据生成索引模式。有关生成索引字段的更多信息，请参见API文档。

## 查询

根据您的用例，有多种方法可以查询 ``Redis`` VectorStore 实现：

- ``similarity_search``: 查找与给定向量最相似的向量。
- ``similarity_search_with_score``: 查找与给定向量最相似的向量并返回向量距离。
- ``similarity_search_limit_score``: 查找与给定向量最相似的向量，并将结果数量限制为 ``score_threshold``。
- ``similarity_search_with_relevance_scores``: 查找与给定向量最相似的向量并返回向量相似度。
- ``max_marginal_relevance_search``: 查找与给定向量最相似的向量，同时优化多样性。

```python
results = rds.similarity_search("foo")
print(results[0].page_content)
```
```output
foo
```

```python
# 返回元数据
results = rds.similarity_search("foo", k=3)
meta = results[1].metadata
print("Redis 中文档的键: ", meta.pop("id"))
print("文档的元数据: ", meta)
```
```output
Redis 中文档的键:  doc:users:a70ca43b3a4e4168bae57c78753a200f
文档的元数据:  {'user': 'derrick', 'job': 'doctor', 'credit_score': 'low', 'age': '45'}
```

```python
# 带分数（距离）
results = rds.similarity_search_with_score("foo", k=5)
for result in results:
    print(f"内容: {result[0].page_content} --- 分数: {result[1]}")
```
```output
内容: foo --- 分数: 0.0
内容: foo --- 分数: 0.0
内容: foo --- 分数: 0.0
内容: bar --- 分数: 0.1566
内容: bar --- 分数: 0.1566
```

```python
# 限制可以返回的向量距离
results = rds.similarity_search_with_score("foo", k=5, distance_threshold=0.1)
for result in results:
    print(f"内容: {result[0].page_content} --- 分数: {result[1]}")
```
```output
内容: foo --- 分数: 0.0
内容: foo --- 分数: 0.0
内容: foo --- 分数: 0.0
```

```python
# 带分数
results = rds.similarity_search_with_relevance_scores("foo", k=5)
for result in results:
    print(f"内容: {result[0].page_content} --- 相似度: {result[1]}")
```
```output
内容: foo --- 相似度: 1.0
内容: foo --- 相似度: 1.0
内容: foo --- 相似度: 1.0
内容: bar --- 相似度: 0.8434
内容: bar --- 相似度: 0.8434
```

```python
# 限制分数（相似度必须超过 .9）
results = rds.similarity_search_with_relevance_scores("foo", k=5, score_threshold=0.9)
for result in results:
    print(f"内容: {result[0].page_content} --- 相似度: {result[1]}")
```
```output
内容: foo --- 相似度: 1.0
内容: foo --- 相似度: 1.0
内容: foo --- 相似度: 1.0
```

```python
# 您还可以按如下方式添加新文档
new_document = ["baz"]
new_metadata = [{"user": "sam", "age": 50, "job": "janitor", "credit_score": "high"}]
# 文档和元数据都必须是列表
rds.add_texts(new_document, new_metadata)
```

```output
['doc:users:b9c71d62a0a34241a37950b448dafd38']
```

```python
# 现在查询新文档
results = rds.similarity_search("baz", k=3)
print(results[0].metadata)
```
```output
{'id': 'doc:users:b9c71d62a0a34241a37950b448dafd38', 'user': 'sam', 'job': 'janitor', 'credit_score': 'high', 'age': '50'}
```

```python
# 使用最大边际相关性搜索来多样化结果
results = rds.max_marginal_relevance_search("foo")
```

```python
# lambda_mult 参数控制结果的多样性，值越低越多样
results = rds.max_marginal_relevance_search("foo", lambda_mult=0.1)
```

## 连接到现有索引

为了在使用 ``Redis`` VectorStore 时索引相同的元数据，您需要传递相同的 ``index_schema``，可以作为 yaml 文件的路径或字典传入。以下展示了如何从索引中获取模式并连接到现有索引。


```python
# write the schema to a yaml file
rds.write_schema("redis_schema.yaml")
```

此示例的模式文件应如下所示：

```yaml
numeric:
- name: age
  no_index: false
  sortable: false
text:
- name: user
  no_index: false
  no_stem: false
  sortable: false
  weight: 1
  withsuffixtrie: false
- name: job
  no_index: false
  no_stem: false
  sortable: false
  weight: 1
  withsuffixtrie: false
- name: credit_score
  no_index: false
  no_stem: false
  sortable: false
  weight: 1
  withsuffixtrie: false
- name: content
  no_index: false
  no_stem: false
  sortable: false
  weight: 1
  withsuffixtrie: false
vector:
- algorithm: FLAT
  block_size: 1000
  datatype: FLOAT32
  dims: 1536
  distance_metric: COSINE
  initial_cap: 20000
  name: content_vector
```

**注意**，这包括 **所有** 可能的模式字段。您可以删除不需要的字段。


```python
# now we can connect to our existing index as follows

new_rds = Redis.from_existing_index(
    embeddings,
    index_name="users",
    redis_url="redis://localhost:6379",
    schema="redis_schema.yaml",
)
results = new_rds.similarity_search("foo", k=3)
print(results[0].metadata)
```
```output
{'id': 'doc:users:8484c48a032d4c4cbe3cc2ed6845fabb', 'user': 'john', 'job': 'engineer', 'credit_score': 'high', 'age': '18'}
```

```python
# see the schemas are the same
new_rds.schema == rds.schema
```



```output
True
```

## 自定义元数据索引

在某些情况下，您可能希望控制元数据映射到哪些字段。例如，您可能希望将 ``credit_score`` 字段设置为分类字段，而不是文本字段（这是所有字符串字段的默认行为）。在这种情况下，您可以在上述每个初始化方法中使用 ``index_schema`` 参数来指定索引的模式。自定义索引模式可以作为字典传递，也可以作为 YAML 文件的路径传递。

模式中的所有参数都有默认值，除了名称，因此您只需指定想要更改的字段。所有名称对应于您在命令行使用 ``redis-cli`` 或在 ``redis-py`` 中使用的参数的小写蛇形版本。有关每个字段参数的更多信息，请参见 [文档](https://redis.io/docs/interact/search-and-query/basic-constructs/field-and-type-options/)

下面的示例展示了如何将 ``credit_score`` 字段指定为标签（分类）字段，而不是文本字段。

```yaml
# index_schema.yml
tag:
    - name: credit_score
text:
    - name: user
    - name: job
numeric:
    - name: age
```

在 Python 中，这看起来像：

```python

index_schema = {
    "tag": [{"name": "credit_score"}],
    "text": [{"name": "user"}, {"name": "job"}],
    "numeric": [{"name": "age"}],
}

```

请注意，仅需要指定 ``name`` 字段。所有其他字段都有默认值。

```python
# 使用上述定义的新模式创建新索引
index_schema = {
    "tag": [{"name": "credit_score"}],
    "text": [{"name": "user"}, {"name": "job"}],
    "numeric": [{"name": "age"}],
}

rds, keys = Redis.from_texts_return_keys(
    texts,
    embeddings,
    metadatas=metadata,
    redis_url="redis://localhost:6379",
    index_name="users_modified",
    index_schema=index_schema,  # 传入新的索引模式
)
```
```output
`index_schema` does not match generated metadata schema.
If you meant to manually override the schema, please ignore this message.
index_schema: {'tag': [{'name': 'credit_score'}], 'text': [{'name': 'user'}, {'name': 'job'}], 'numeric': [{'name': 'age'}]}
generated_schema: {'text': [{'name': 'user'}, {'name': 'job'}, {'name': 'credit_score'}], 'numeric': [{'name': 'age'}], 'tag': []}
```
上述警告旨在通知用户何时覆盖默认行为。如果您是故意覆盖该行为，请忽略它。

## 混合过滤

通过内置于 LangChain 的 Redis 过滤表达式语言，您可以创建任意长度的混合过滤链，以用于过滤搜索结果。该表达式语言源自 [RedisVL 表达式语法](https://redisvl.com)，旨在易于使用和理解。

以下是可用的过滤类型：
- ``RedisText``：通过对元数据字段进行全文搜索进行过滤。支持精确匹配、模糊匹配和通配符匹配。
- ``RedisNum``：通过对元数据字段进行数值范围过滤。
- ``RedisTag``：通过对基于字符串的分类元数据字段进行精确匹配进行过滤。可以指定多个标签，例如 "tag1,tag2,tag3"。

以下是利用这些过滤器的示例。

```python

from langchain_community.vectorstores.redis import RedisText, RedisNum, RedisTag

# 精确匹配
has_high_credit = RedisTag("credit_score") == "high"
does_not_have_high_credit = RedisTag("credit_score") != "low"

# 模糊匹配
job_starts_with_eng = RedisText("job") % "eng*"
job_is_engineer = RedisText("job") == "engineer"
job_is_not_engineer = RedisText("job") != "engineer"

# 数值过滤
age_is_18 = RedisNum("age") == 18
age_is_not_18 = RedisNum("age") != 18
age_is_greater_than_18 = RedisNum("age") > 18
age_is_less_than_18 = RedisNum("age") < 18
age_is_greater_than_or_equal_to_18 = RedisNum("age") >= 18
age_is_less_than_or_equal_to_18 = RedisNum("age") <= 18

```

``RedisFilter`` 类可用于简化这些过滤器的导入，如下所示

```python

from langchain_community.vectorstores.redis import RedisFilter

# 与上述示例相同
has_high_credit = RedisFilter.tag("credit_score") == "high"
does_not_have_high_credit = RedisFilter.num("age") > 8
job_starts_with_eng = RedisFilter.text("job") % "eng*"
```

以下是使用混合过滤进行搜索的示例

```python
from langchain_community.vectorstores.redis import RedisText

is_engineer = RedisText("job") == "engineer"
results = rds.similarity_search("foo", k=3, filter=is_engineer)

print("Job:", results[0].metadata["job"])
print("Engineers in the dataset:", len(results))
```
```output
Job: engineer
Engineers in the dataset: 2
```

```python
# 模糊匹配
starts_with_doc = RedisText("job") % "doc*"
results = rds.similarity_search("foo", k=3, filter=starts_with_doc)

for result in results:
    print("Job:", result.metadata["job"])
print("Jobs in dataset that start with 'doc':", len(results))
```
```output
Job: doctor
Job: doctor
Jobs in dataset that start with 'doc': 2
```

```python
from langchain_community.vectorstores.redis import RedisNum

is_over_18 = RedisNum("age") > 18
is_under_99 = RedisNum("age") < 99
age_range = is_over_18 & is_under_99
results = rds.similarity_search("foo", filter=age_range)

for result in results:
    print("User:", result.metadata["user"], "is", result.metadata["age"])
```
```output
User: derrick is 45
User: nancy is 94
User: joe is 35
```

```python
# 确保在构造过滤表达式时使用括号
# 如果在初始化时进行构造
age_range = (RedisNum("age") > 18) & (RedisNum("age") < 99)
results = rds.similarity_search("foo", filter=age_range)

for result in results:
    print("User:", result.metadata["user"], "is", result.metadata["age"])
```
```output
User: derrick is 45
User: nancy is 94
User: joe is 35
```

## Redis 作为检索器

在这里，我们讨论使用向量存储作为检索器的不同选项。

我们可以使用三种不同的搜索方法进行检索。默认情况下，它将使用语义相似性。

```python
query = "foo"
results = rds.similarity_search_with_score(query, k=3, return_metadata=True)

for result in results:
    print("Content:", result[0].page_content, " --- Score: ", result[1])
```
```output
Content: foo  --- Score:  0.0
Content: foo  --- Score:  0.0
Content: foo  --- Score:  0.0
```

```python
retriever = rds.as_retriever(search_type="similarity", search_kwargs={"k": 4})
```

```python
docs = retriever.invoke(query)
docs
```

```output
[Document(page_content='foo', metadata={'id': 'doc:users_modified:988ecca7574048e396756efc0e79aeca', 'user': 'john', 'job': 'engineer', 'credit_score': 'high', 'age': '18'}),
 Document(page_content='foo', metadata={'id': 'doc:users_modified:009b1afeb4084cc6bdef858c7a99b48e', 'user': 'derrick', 'job': 'doctor', 'credit_score': 'low', 'age': '45'}),
 Document(page_content='foo', metadata={'id': 'doc:users_modified:7087cee9be5b4eca93c30fbdd09a2731', 'user': 'nancy', 'job': 'doctor', 'credit_score': 'high', 'age': '94'}),
 Document(page_content='bar', metadata={'id': 'doc:users_modified:01ef6caac12b42c28ad870aefe574253', 'user': 'tyler', 'job': 'engineer', 'credit_score': 'high', 'age': '100'})]
```

还有 `similarity_distance_threshold` 检索器，允许用户指定向量距离。

```python
retriever = rds.as_retriever(
    search_type="similarity_distance_threshold",
    search_kwargs={"k": 4, "distance_threshold": 0.1},
)
```

```python
docs = retriever.invoke(query)
docs
```

```output
[Document(page_content='foo', metadata={'id': 'doc:users_modified:988ecca7574048e396756efc0e79aeca', 'user': 'john', 'job': 'engineer', 'credit_score': 'high', 'age': '18'}),
 Document(page_content='foo', metadata={'id': 'doc:users_modified:009b1afeb4084cc6bdef858c7a99b48e', 'user': 'derrick', 'job': 'doctor', 'credit_score': 'low', 'age': '45'}),
 Document(page_content='foo', metadata={'id': 'doc:users_modified:7087cee9be5b4eca93c30fbdd09a2731', 'user': 'nancy', 'job': 'doctor', 'credit_score': 'high', 'age': '94'})]
```

最后，`similarity_score_threshold` 允许用户定义相似文档的最低分数。

```python
retriever = rds.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={"score_threshold": 0.9, "k": 10},
)
```

```python
retriever.invoke("foo")
```

```output
[Document(page_content='foo', metadata={'id': 'doc:users_modified:988ecca7574048e396756efc0e79aeca', 'user': 'john', 'job': 'engineer', 'credit_score': 'high', 'age': '18'}),
 Document(page_content='foo', metadata={'id': 'doc:users_modified:009b1afeb4084cc6bdef858c7a99b48e', 'user': 'derrick', 'job': 'doctor', 'credit_score': 'low', 'age': '45'}),
 Document(page_content='foo', metadata={'id': 'doc:users_modified:7087cee9be5b4eca93c30fbdd09a2731', 'user': 'nancy', 'job': 'doctor', 'credit_score': 'high', 'age': '94'})]
```

```python
retriever = rds.as_retriever(
    search_type="mmr", search_kwargs={"fetch_k": 20, "k": 4, "lambda_mult": 0.1}
)
```

```python
retriever.invoke("foo")
```

```output
[Document(page_content='foo', metadata={'id': 'doc:users:8f6b673b390647809d510112cde01a27', 'user': 'john', 'job': 'engineer', 'credit_score': 'high', 'age': '18'}),
 Document(page_content='bar', metadata={'id': 'doc:users:93521560735d42328b48c9c6f6418d6a', 'user': 'tyler', 'job': 'engineer', 'credit_score': 'high', 'age': '100'}),
 Document(page_content='foo', metadata={'id': 'doc:users:125ecd39d07845eabf1a699d44134a5b', 'user': 'nancy', 'job': 'doctor', 'credit_score': 'high', 'age': '94'}),
 Document(page_content='foo', metadata={'id': 'doc:users:d6200ab3764c466082fde3eaab972a2a', 'user': 'derrick', 'job': 'doctor', 'credit_score': 'low', 'age': '45'})]
```

## 删除键和索引

要删除您的条目，您必须通过它们的键来访问它们。

```python
Redis.delete(keys, redis_url="redis://localhost:6379")
```

```output
True
```

```python
# 也删除索引
Redis.drop_index(
    index_name="users", delete_documents=True, redis_url="redis://localhost:6379"
)
Redis.drop_index(
    index_name="users_modified",
    delete_documents=True,
    redis_url="redis://localhost:6379",
)
```

```output
True
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)