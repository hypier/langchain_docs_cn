---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/elasticsearch.ipynb
---

# Elasticsearch

>[Elasticsearch](https://www.elastic.co/elasticsearch/) 是一个分布式的、基于REST的搜索和分析引擎，能够执行向量和词汇搜索。它建立在Apache Lucene库之上。

本笔记本展示了如何使用与`Elasticsearch`数据库相关的功能。

```python
%pip install --upgrade --quiet langchain-elasticsearch langchain-openai tiktoken langchain
```

## 运行和连接到Elasticsearch

有两种主要方法可以设置Elasticsearch实例以供使用：

1. Elastic Cloud：Elastic Cloud是一个托管的Elasticsearch服务。注册以获取[免费试用](https://cloud.elastic.co/registration?utm_source=langchain&utm_content=documentation)。

要连接到一个不需要登录凭据的Elasticsearch实例（以安全模式启动docker实例），请将Elasticsearch URL和索引名称与嵌入对象一起传递给构造函数。

2. 本地安装Elasticsearch：通过在本地运行Elasticsearch来开始使用。最简单的方法是使用官方的Elasticsearch Docker镜像。有关更多信息，请参见[Elasticsearch Docker文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)。

### 通过 Docker 运行 Elasticsearch
示例：运行一个单节点的 Elasticsearch 实例，安全性已禁用。这不建议用于生产环境。

```bash
docker run -p 9200:9200 -e "discovery.type=single-node" -e "xpack.security.enabled=false" -e "xpack.security.http.ssl.enabled=false" docker.elastic.co/elasticsearch/elasticsearch:8.12.1
```

一旦 Elasticsearch 实例运行起来，您可以使用 Elasticsearch URL 和索引名称以及嵌入对象连接到构造函数。

示例：
```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings

embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    es_url="http://localhost:9200",
    index_name="test_index",
    embedding=embedding
)
```

### 认证
对于生产环境，我们建议您启用安全功能。要使用登录凭据进行连接，您可以使用参数 `es_api_key` 或 `es_user` 和 `es_password`。

示例：
```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings

embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    es_url="http://localhost:9200",
    index_name="test_index",
    embedding=embedding,
    es_user="elastic",
    es_password="changeme"
)
```

您还可以使用 `Elasticsearch` 客户端对象，这样可以提供更多灵活性，例如配置最大重试次数。

示例：
```python
import elasticsearch
from langchain_elasticsearch import ElasticsearchStore

es_client= elasticsearch.Elasticsearch(
    hosts=["http://localhost:9200"],
    es_user="elastic",
    es_password="changeme"
    max_retries=10,
)

embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    index_name="test_index",
    es_connection=es_client,
    embedding=embedding,
)
```

#### 如何获取默认“elastic”用户的密码？

要获取默认“elastic”用户的 Elastic Cloud 密码：
1. 登录到 Elastic Cloud 控制台 https://cloud.elastic.co
2. 转到“安全” > “用户”
3. 找到“elastic”用户并点击“编辑”
4. 点击“重置密码”
5. 按照提示重置密码

#### 如何获取 API 密钥？

要获取 API 密钥：
1. 登录到 Elastic Cloud 控制台 https://cloud.elastic.co
2. 打开 Kibana 并转到 Stack Management > API Keys
3. 点击“创建 API 密钥”
4. 输入 API 密钥的名称并点击“创建”
5. 复制 API 密钥并将其粘贴到 `api_key` 参数中

### Elastic Cloud
要连接到 Elastic Cloud 上的 Elasticsearch 实例，您可以使用 `es_cloud_id` 参数或 `es_url`。

示例：
```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings

embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    es_cloud_id="<cloud_id>",
    index_name="test_index",
    embedding=embedding,
    es_user="elastic",
    es_password="changeme"
)
```

要使用 `OpenAIEmbeddings`，我们必须在环境中配置 OpenAI API 密钥。

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

## 基本示例
在这个示例中，我们将通过 TextLoader 加载 "state_of_the_union.txt"，将文本分成 500 字的块，然后将每个块索引到 Elasticsearch 中。

数据索引完成后，我们执行一个简单的查询，以查找与查询 "总统对 Ketanji Brown Jackson 说了什么" 最相似的前 4 个块。

Elasticsearch 在 localhost:9200 上本地运行，使用 [docker](#running-elasticsearch-via-docker)。有关如何从 Elastic Cloud 连接到 Elasticsearch 的更多详细信息，请参见上面的 [连接与身份验证](#authentication)。

```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```

```python
db = ElasticsearchStore.from_documents(
    docs,
    embeddings,
    es_url="http://localhost:9200",
    index_name="test-basic",
)

db.client.indices.refresh(index="test-basic")

query = "What did the president say about Ketanji Brown Jackson"
results = db.similarity_search(query)
print(results)
```
```output
[Document(page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../how_to/state_of_the_union.txt'}), Document(page_content='As I said last year, especially to our younger transgender Americans, I will always have your back as your President, so you can be yourself and reach your God-given potential. \n\nWhile it often appears that we never agree, that isn’t true. I signed 80 bipartisan bills into law last year. From preventing government shutdowns to protecting Asian-Americans from still-too-common hate crimes to reforming military justice.', metadata={'source': '../../how_to/state_of_the_union.txt'}), Document(page_content='A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. \n\nAnd if we are to advance liberty and justice, we need to secure the Border and fix the immigration system.', metadata={'source': '../../how_to/state_of_the_union.txt'}), Document(page_content='This is personal to me and Jill, to Kamala, and to so many of you. \n\nCancer is the #2 cause of death in America–second only to heart disease. \n\nLast month, I announced our plan to supercharge  \nthe Cancer Moonshot that President Obama asked me to lead six years ago. \n\nOur goal is to cut the cancer death rate by at least 50% over the next 25 years, turn more cancers from death sentences into treatable diseases.  \n\nMore support for patients and families.', metadata={'source': '../../how_to/state_of_the_union.txt'})]
```

# 元数据

`ElasticsearchStore` 支持与文档一起存储的元数据。这个元数据字典对象存储在 Elasticsearch 文档中的元数据对象字段中。根据元数据值，Elasticsearch 会通过推断元数据值的数据类型自动设置映射。例如，如果元数据值是字符串，Elasticsearch 会将元数据对象字段的映射设置为字符串类型。

```python
# Adding metadata to documents
for i, doc in enumerate(docs):
    doc.metadata["date"] = f"{range(2010, 2020)[i % 10]}-01-01"
    doc.metadata["rating"] = range(1, 6)[i % 5]
    doc.metadata["author"] = ["John Doe", "Jane Doe"][i % 2]

db = ElasticsearchStore.from_documents(
    docs, embeddings, es_url="http://localhost:9200", index_name="test-metadata"
)

query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
print(docs[0].metadata)
```
```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2016-01-01', 'rating': 2, 'author': 'John Doe'}
```

## 过滤元数据
通过在文档中添加元数据，您可以在查询时添加元数据过滤。

### 示例：按精确关键词过滤
注意：我们使用的是未分析的关键词子字段


```python
docs = db.similarity_search(
    query, filter=[{"term": {"metadata.author.keyword": "John Doe"}}]
)
print(docs[0].metadata)
```
```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2016-01-01', 'rating': 2, 'author': 'John Doe'}
```

### 示例：按部分匹配过滤
本示例展示了如何按部分匹配进行过滤。当您不知道元数据字段的确切值时，这非常有用。例如，如果您想按元数据字段 `author` 进行过滤，但不知道作者的确切值，您可以使用部分匹配按作者的姓进行过滤。也支持模糊匹配。

"Jon" 与 "John Doe" 匹配，因为 "Jon" 是与 "John" 令牌的近似匹配。

```python
docs = db.similarity_search(
    query,
    filter=[{"match": {"metadata.author": {"query": "Jon", "fuzziness": "AUTO"}}}],
)
print(docs[0].metadata)
```
```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2016-01-01', 'rating': 2, 'author': 'John Doe'}
```

### 示例：按日期范围过滤

```python
docs = db.similarity_search(
    "Any mention about Fred?",
    filter=[{"range": {"metadata.date": {"gte": "2010-01-01"}}}],
)
print(docs[0].metadata)
```
```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2012-01-01', 'rating': 3, 'author': 'John Doe', 'geo_location': {'lat': 40.12, 'lon': -71.34}}
```

### 示例：按数值范围过滤

```python
docs = db.similarity_search(
    "Any mention about Fred?", filter=[{"range": {"metadata.rating": {"gte": 2}}}]
)
print(docs[0].metadata)
```
```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2012-01-01', 'rating': 3, 'author': 'John Doe', 'geo_location': {'lat': 40.12, 'lon': -71.34}}
```

### 示例：按地理距离过滤
需要为 `metadata.geo_location` 声明一个带有 geo_point 映射的索引。

```python
docs = db.similarity_search(
    "Any mention about Fred?",
    filter=[
        {
            "geo_distance": {
                "distance": "200km",
                "metadata.geo_location": {"lat": 40, "lon": -70},
            }
        }
    ],
)
print(docs[0].metadata)
```

过滤器支持比上述更多类型的查询。

在 [文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) 中了解更多信息。

# 距离相似度算法
Elasticsearch 支持以下向量距离相似度算法：

- cosine
- euclidean
- dot_product

cosine 相似度算法是默认的。

您可以通过 similarity 参数指定所需的相似度算法。

**注意**
根据检索策略，相似度算法在查询时无法更改。它需要在创建字段的索引映射时设置。如果您需要更改相似度算法，您需要删除索引并使用正确的 distance_strategy 重新创建它。

```python

db = ElasticsearchStore.from_documents(
    docs, 
    embeddings, 
    es_url="http://localhost:9200", 
    index_name="test",
    distance_strategy="COSINE"
    # distance_strategy="EUCLIDEAN_DISTANCE"
    # distance_strategy="DOT_PRODUCT"
)

```

# 检索策略
Elasticsearch 相较于其他仅支持向量的数据库具有很大的优势，因为它能够支持广泛的检索策略。在本笔记本中，我们将配置 `ElasticsearchStore` 以支持一些最常见的检索策略。

默认情况下，`ElasticsearchStore` 使用 `DenseVectorStrategy`（在 0.2.0 版本之前称为 `ApproxRetrievalStrategy`）。

## DenseVectorStrategy
这将返回与查询向量最相似的前 `k` 个向量。 `k` 参数在初始化 `ElasticsearchStore` 时设置。 默认值为 `10`。

```python
from langchain_elasticsearch import DenseVectorStrategy

db = ElasticsearchStore.from_documents(
    docs,
    embeddings,
    es_url="http://localhost:9200",
    index_name="test",
    strategy=DenseVectorStrategy(),
)

docs = db.similarity_search(
    query="What did the president say about Ketanji Brown Jackson?", k=10
)
```

### 示例：使用密集向量和关键词搜索的混合检索
本示例将演示如何配置 `ElasticsearchStore` 以执行混合检索，使用近似语义搜索和基于关键词的搜索的组合。

我们使用 RRF 来平衡来自不同检索方法的两个分数。

要启用混合检索，我们需要在 `DenseVectorStrategy` 构造函数中设置 `hybrid=True`。

```python

db = ElasticsearchStore.from_documents(
    docs, 
    embeddings, 
    es_url="http://localhost:9200", 
    index_name="test",
    strategy=DenseVectorStrategy(hybrid=True)
)
```

当启用 `hybrid` 时，执行的查询将是近似语义搜索和基于关键词的搜索的组合。

它将使用 `rrf`（互惠排名融合）来平衡来自不同检索方法的两个分数。

**注意** RRF 需要 Elasticsearch 8.9.0 或更高版本。

```json
{
    "knn": {
        "field": "vector",
        "filter": [],
        "k": 1,
        "num_candidates": 50,
        "query_vector": [1.0, ..., 0.0],
    },
    "query": {
        "bool": {
            "filter": [],
            "must": [{"match": {"text": {"query": "foo"}}}],
        }
    },
    "rank": {"rrf": {}},
}
```

### 示例：在Elasticsearch中使用嵌入模型进行稠密向量搜索
本示例将展示如何配置`ElasticsearchStore`以使用在Elasticsearch中部署的嵌入模型进行稠密向量检索。

要使用此功能，请通过`query_model_id`参数在`DenseVectorStrategy`构造函数中指定model_id。

**注意** 这要求模型已在Elasticsearch ml节点中部署并运行。请参阅[notebook示例](https://github.com/elastic/elasticsearch-labs/blob/main/notebooks/integrations/hugging-face/loading-model-from-hugging-face.ipynb)，了解如何使用eland部署模型。



```python
DENSE_SELF_DEPLOYED_INDEX_NAME = "test-dense-self-deployed"

# Note: This does not have an embedding function specified
# Instead, we will use the embedding model deployed in Elasticsearch
db = ElasticsearchStore(
    es_cloud_id="<your cloud id>",
    es_user="elastic",
    es_password="<your password>",
    index_name=DENSE_SELF_DEPLOYED_INDEX_NAME,
    query_field="text_field",
    vector_query_field="vector_query_field.predicted_value",
    strategy=DenseVectorStrategy(model_id="sentence-transformers__all-minilm-l6-v2"),
)

# Setup a Ingest Pipeline to perform the embedding
# of the text field
db.client.ingest.put_pipeline(
    id="test_pipeline",
    processors=[
        {
            "inference": {
                "model_id": "sentence-transformers__all-minilm-l6-v2",
                "field_map": {"query_field": "text_field"},
                "target_field": "vector_query_field",
            }
        }
    ],
)

# creating a new index with the pipeline,
# not relying on langchain to create the index
db.client.indices.create(
    index=DENSE_SELF_DEPLOYED_INDEX_NAME,
    mappings={
        "properties": {
            "text_field": {"type": "text"},
            "vector_query_field": {
                "properties": {
                    "predicted_value": {
                        "type": "dense_vector",
                        "dims": 384,
                        "index": True,
                        "similarity": "l2_norm",
                    }
                }
            },
        }
    },
    settings={"index": {"default_pipeline": "test_pipeline"}},
)

db.from_texts(
    ["hello world"],
    es_cloud_id="<cloud id>",
    es_user="elastic",
    es_password="<cloud password>",
    index_name=DENSE_SELF_DEPLOYED_INDEX_NAME,
    query_field="text_field",
    vector_query_field="vector_query_field.predicted_value",
    strategy=DenseVectorStrategy(model_id="sentence-transformers__all-minilm-l6-v2"),
)

# Perform search
db.similarity_search("hello world", k=10)
```

## SparseVectorStrategy (ELSER)
该策略使用Elasticsearch的稀疏向量检索来获取前k个结果。目前我们仅支持我们自己的“ELSER”嵌入模型。

**注意** 这需要在Elasticsearch ml节点中部署并运行ELSER模型。

要使用此功能，请在`ElasticsearchStore`构造函数中指定`SparseVectorStrategy`（在0.2.0版本之前称为`SparseVectorRetrievalStrategy`）。您需要提供模型ID。

```python
from langchain_elasticsearch import SparseVectorStrategy

# Note that this example doesn't have an embedding function. This is because we infer the tokens at index time and at query time within Elasticsearch.
# This requires the ELSER model to be loaded and running in Elasticsearch.
db = ElasticsearchStore.from_documents(
    docs,
    es_cloud_id="<cloud id>",
    es_user="elastic",
    es_password="<cloud password>",
    index_name="test-elser",
    strategy=SparseVectorStrategy(model_id=".elser_model_2"),
)

db.client.indices.refresh(index="test-elser")

results = db.similarity_search(
    "What did the president say about Ketanji Brown Jackson", k=4
)
print(results[0])
```
```output
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
```

## DenseVectorScriptScoreStrategy
该策略使用Elasticsearch的脚本评分查询来执行精确向量检索（也称为暴力检索），以获取前k个结果。（在版本0.2.0之前，该策略被称为`ExactRetrievalStrategy`。）

要使用此策略，请在`ElasticsearchStore`构造函数中指定`DenseVectorScriptScoreStrategy`。

```python
from langchain_elasticsearch import SparseVectorStrategy

db = ElasticsearchStore.from_documents(
    docs, 
    embeddings, 
    es_url="http://localhost:9200", 
    index_name="test",
    strategy=DenseVectorScriptScoreStrategy(),
)
```

## BM25Strategy
最后，您可以使用全文关键字搜索。

要使用此功能，请在 `ElasticsearchStore` 构造函数中指定 `BM25Strategy`。

```python
from langchain_elasticsearch import BM25Strategy

db = ElasticsearchStore.from_documents(
    docs, 
    es_url="http://localhost:9200", 
    index_name="test",
    strategy=BM25Strategy(),
)
```

## BM25检索策略
该策略允许用户仅使用纯BM25进行搜索，而不使用向量搜索。

要使用此功能，请在`ElasticsearchStore`构造函数中指定`BM25RetrievalStrategy`。

请注意，在下面的示例中，没有指定嵌入选项，这表明搜索是在不使用嵌入的情况下进行的。


```python
from langchain_elasticsearch import ElasticsearchStore

db = ElasticsearchStore(
    es_url="http://localhost:9200",
    index_name="test_index",
    strategy=ElasticsearchStore.BM25RetrievalStrategy(),
)

db.add_texts(
    ["foo", "foo bar", "foo bar baz", "bar", "bar baz", "baz"],
)

results = db.similarity_search(query="foo", k=10)
print(results)
```
```output
[Document(page_content='foo'), Document(page_content='foo bar'), Document(page_content='foo bar baz')]
```

## 自定义查询
通过搜索中的 `custom_query` 参数，您可以调整用于从 Elasticsearch 检索文档的查询。这在您想要使用更复杂的查询以支持字段的线性提升时非常有用。

```python
# Example of a custom query thats just doing a BM25 search on the text field.
def custom_query(query_body: dict, query: str):
    """Custom query to be used in Elasticsearch.
    Args:
        query_body (dict): Elasticsearch query body.
        query (str): Query string.
    Returns:
        dict: Elasticsearch query body.
    """
    print("Query Retriever created by the retrieval strategy:")
    print(query_body)
    print()

    new_query_body = {"query": {"match": {"text": query}}}

    print("Query thats actually used in Elasticsearch:")
    print(new_query_body)
    print()

    return new_query_body


results = db.similarity_search(
    "What did the president say about Ketanji Brown Jackson",
    k=4,
    custom_query=custom_query,
)
print("Results:")
print(results[0])
```
```output
Query Retriever created by the retrieval strategy:
{'query': {'bool': {'must': [{'text_expansion': {'vector.tokens': {'model_id': '.elser_model_1', 'model_text': 'What did the president say about Ketanji Brown Jackson'}}}], 'filter': []}}}

Query thats actually used in Elasticsearch:
{'query': {'match': {'text': 'What did the president say about Ketanji Brown Jackson'}}}

Results:
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
```

# 自定义文档构建器

通过在搜索中使用 ```doc_builder``` 参数，您可以调整如何使用从 Elasticsearch 检索的数据构建文档。如果您有未使用 Langchain 创建的索引，这尤其有用。

```python
from typing import Dict

from langchain_core.documents import Document


def custom_document_builder(hit: Dict) -> Document:
    src = hit.get("_source", {})
    return Document(
        page_content=src.get("content", "Missing content!"),
        metadata={
            "page_number": src.get("page_number", -1),
            "original_filename": src.get("original_filename", "Missing filename!"),
        },
    )


results = db.similarity_search(
    "What did the president say about Ketanji Brown Jackson",
    k=4,
    doc_builder=custom_document_builder,
)
print("Results:")
print(results[0])
```

# 常见问题解答

## 问题：我在将文档索引到Elasticsearch时遇到超时错误。该如何解决？
一个可能的问题是您的文档在索引到Elasticsearch时可能需要更长的时间。ElasticsearchStore使用Elasticsearch批量API，该API有一些默认设置，您可以调整这些设置以减少超时错误的可能性。

当您使用SparseVectorRetrievalStrategy时，这也是一个不错的主意。

默认设置为：
- `chunk_size`: 500
- `max_chunk_bytes`: 100MB

要调整这些设置，您可以将`chunk_size`和`max_chunk_bytes`参数传递给ElasticsearchStore的`add_texts`方法。

```python
    vector_store.add_texts(
        texts,
        bulk_kwargs={
            "chunk_size": 50,
            "max_chunk_bytes": 200000000
        }
    )
```

# 升级到 ElasticsearchStore

如果您已经在基于 langchain 的项目中使用 Elasticsearch，您可能正在使用旧的实现：`ElasticVectorSearch` 和 `ElasticKNNSearch`，这些实现现在已被弃用。我们引入了一种新的实现，称为 `ElasticsearchStore`，它更灵活且更易于使用。此笔记本将指导您完成升级到新实现的过程。

## 有什么新变化？

新的实现现在是一个名为 `ElasticsearchStore` 的类，可用于近似密集向量、精确密集向量、稀疏向量（ELSER）、BM25 检索和混合检索，采用策略。

## 我正在使用 ElasticKNNSearch

旧实现：

```python

from langchain_community.vectorstores.elastic_vector_search import ElasticKNNSearch

db = ElasticKNNSearch(
  elasticsearch_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding
)

```

新实现：

```python

from langchain_elasticsearch import ElasticsearchStore, DenseVectorStrategy

db = ElasticsearchStore(
  es_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding,
  # 如果您使用 model_id
  # strategy=DenseVectorStrategy(model_id="test_model")
  # 如果您使用混合搜索
  # strategy=DenseVectorStrategy(hybrid=True)
)

```

## 我正在使用 ElasticVectorSearch

旧实现：

```python

from langchain_community.vectorstores.elastic_vector_search import ElasticVectorSearch

db = ElasticVectorSearch(
  elasticsearch_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding
)

```

新实现：

```python

from langchain_elasticsearch import ElasticsearchStore, DenseVectorScriptScoreStrategy

db = ElasticsearchStore(
  es_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding,
  strategy=DenseVectorScriptScoreStrategy()
)

```


```python
db.client.indices.delete(
    index="test-metadata, test-elser, test-basic",
    ignore_unavailable=True,
    allow_no_indices=True,
)
```



```output
ObjectApiResponse({'acknowledged': True})
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)