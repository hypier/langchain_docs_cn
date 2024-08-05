---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/couchbase.ipynb
---

# Couchbase 
[Couchbase](http://couchbase.com/) 是一个屡获殊荣的分布式 NoSQL 云数据库，提供无与伦比的多功能性、性能、可扩展性和经济价值，适用于您的所有云、移动、人工智能和边缘计算应用程序。Couchbase 通过为开发人员提供编码辅助和为其应用程序提供向量搜索来拥抱人工智能。

向量搜索是 Couchbase 中的 [全文搜索服务](https://docs.couchbase.com/server/current/learn/services-and-indexes/services/search-service.html)（搜索服务）的一部分。

本教程将解释如何在 Couchbase 中使用向量搜索。您可以使用 [Couchbase Capella](https://www.couchbase.com/products/capella/) 和您自我管理的 Couchbase Server。

## 安装


```python
%pip install --upgrade --quiet langchain langchain-openai langchain-couchbase
```


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

## 导入向量存储和嵌入

```python
from langchain_couchbase.vectorstores import CouchbaseVectorStore
from langchain_openai import OpenAIEmbeddings
```

## 创建 Couchbase 连接对象
我们最初创建一个与 Couchbase 集群的连接，然后将集群对象传递给向量存储。

在这里，我们使用用户名和密码进行连接。您也可以使用任何其他支持的方式连接到您的集群。

有关连接到 Couchbase 集群的更多信息，请查看 [Python SDK 文档](https://docs.couchbase.com/python-sdk/current/hello-world/start-using-sdk.html#connect)。

```python
COUCHBASE_CONNECTION_STRING = (
    "couchbase://localhost"  # 或 "couchbases://localhost" 如果使用 TLS
)
DB_USERNAME = "Administrator"
DB_PASSWORD = "Password"
```

```python
from datetime import timedelta

from couchbase.auth import PasswordAuthenticator
from couchbase.cluster import Cluster
from couchbase.options import ClusterOptions

auth = PasswordAuthenticator(DB_USERNAME, DB_PASSWORD)
options = ClusterOptions(auth)
cluster = Cluster(COUCHBASE_CONNECTION_STRING, options)

# 等待集群准备就绪。
cluster.wait_until_ready(timedelta(seconds=5))
```

我们现在将设置在 Couchbase 集群中用于向量搜索的桶、范围和集合名称。

在本示例中，我们使用默认的范围和集合。

```python
BUCKET_NAME = "testing"
SCOPE_NAME = "_default"
COLLECTION_NAME = "_default"
SEARCH_INDEX_NAME = "vector-index"
```

在本教程中，我们将使用 OpenAI 嵌入。

```python
embeddings = OpenAIEmbeddings()
```

## 创建搜索索引
目前，搜索索引需要通过 Couchbase Capella 或服务器 UI 创建，或者使用 REST 接口。

让我们在测试桶上定义一个名为 `vector-index` 的搜索索引。

在这个例子中，让我们使用 UI 上搜索服务的导入索引功能。

我们正在定义一个索引，位于 `testing` 桶的 `_default` 范围内，针对 `_default` 集合，向量字段设置为 `embedding`，维度为 1536，文本字段设置为 `text`。我们还将文档中 `metadata` 下的所有字段作为动态映射进行索引和存储，以适应不同的文档结构。相似度度量设置为 `dot_product`。

### 如何将索引导入全文搜索服务？
 - [Couchbase Server](https://docs.couchbase.com/server/current/search/import-search-index.html)
     - 点击搜索 -> 添加索引 -> 导入
     - 在导入屏幕中复制以下索引定义
     - 点击创建索引以创建索引。
 - [Couchbase Capella](https://docs.couchbase.com/cloud/search/import-search-index.html)
     - 将索引定义复制到新文件 `index.json`
     - 根据文档中的说明在 Capella 中导入该文件。
     - 点击创建索引以创建索引。

### 索引定义
```
{
 "name": "vector-index",
 "type": "fulltext-index",
 "params": {
  "doc_config": {
   "docid_prefix_delim": "",
   "docid_regexp": "",
   "mode": "type_field",
   "type_field": "type"
  },
  "mapping": {
   "default_analyzer": "standard",
   "default_datetime_parser": "dateTimeOptional",
   "default_field": "_all",
   "default_mapping": {
    "dynamic": true,
    "enabled": true,
    "properties": {
     "metadata": {
      "dynamic": true,
      "enabled": true
     },
     "embedding": {
      "enabled": true,
      "dynamic": false,
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
     "text": {
      "enabled": true,
      "dynamic": false,
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
   },
   "default_type": "_default",
   "docvalues_dynamic": false,
   "index_dynamic": true,
   "store_dynamic": true,
   "type_field": "_type"
  },
  "store": {
   "indexType": "scorch",
   "segmentVersion": 16
  }
 },
 "sourceType": "gocbcore",
 "sourceName": "testing",
 "sourceParams": {},
 "planParams": {
  "maxPartitionsPerPIndex": 103,
  "indexPartitions": 10,
  "numReplicas": 0
 }
}
```

有关如何创建支持向量字段的搜索索引的更多详细信息，请参阅文档。

- [Couchbase Capella](https://docs.couchbase.com/cloud/vector-search/create-vector-search-index-ui.html)

- [Couchbase Server](https://docs.couchbase.com/server/current/vector-search/create-vector-search-index-ui.html)

## 创建向量存储
我们使用集群信息和搜索索引名称创建向量存储对象。

```python
vector_store = CouchbaseVectorStore(
    cluster=cluster,
    bucket_name=BUCKET_NAME,
    scope_name=SCOPE_NAME,
    collection_name=COLLECTION_NAME,
    embedding=embeddings,
    index_name=SEARCH_INDEX_NAME,
)
```

### 指定文本和嵌入字段
您可以选择性地使用 `text_key` 和 `embedding_key` 字段为文档指定文本和嵌入字段。
```
vector_store = CouchbaseVectorStore(
    cluster=cluster,
    bucket_name=BUCKET_NAME,
    scope_name=SCOPE_NAME,
    collection_name=COLLECTION_NAME,
    embedding=embeddings,
    index_name=SEARCH_INDEX_NAME,
    text_key="text",
    embedding_key="embedding",
)
```

## 基本向量搜索示例
在这个示例中，我们将通过 TextLoader 加载 "state_of_the_union.txt" 文件，将文本分割成 500 个字符的块，不重叠，并将所有这些块索引到 Couchbase 中。

数据索引后，我们执行一个简单的查询，以找到与查询 "总统对 Ketanji Brown Jackson 的评论是什么" 相似的前 4 个块。

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```

```python
vector_store = CouchbaseVectorStore.from_documents(
    documents=docs,
    embedding=embeddings,
    cluster=cluster,
    bucket_name=BUCKET_NAME,
    scope_name=SCOPE_NAME,
    collection_name=COLLECTION_NAME,
    index_name=SEARCH_INDEX_NAME,
)
```

```python
query = "What did president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(query)
print(results[0])
```
```output
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
```

## 带分数的相似性搜索
您可以通过调用 `similarity_search_with_score` 方法来获取结果的分数。

```python
query = "What did president say about Ketanji Brown Jackson"
results = vector_store.similarity_search_with_score(query)
document, score = results[0]
print(document)
print(f"Score: {score}")
```
```output
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
Score: 0.8211871385574341
```

## 指定返回字段
您可以使用 `fields` 参数在搜索中指定要从文档中返回的字段。这些字段作为返回文档的 `metadata` 对象的一部分返回。您可以提取存储在搜索索引中的任何字段。文档的 `text_key` 作为文档的 `page_content` 的一部分返回。

如果您未指定要提取的字段，则返回索引中存储的所有字段。

如果您想提取元数据中的某个字段，则需要使用 `.` 指定它。

例如，要提取元数据中的 `source` 字段，您需要指定 `metadata.source`。



```python
query = "What did president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(query, fields=["metadata.source"])
print(results[0])
```
```output
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
```

## 混合搜索
Couchbase 允许通过将向量搜索结果与文档中非向量字段的搜索（如 `metadata` 对象）相结合来进行混合搜索。

结果将基于向量搜索和搜索服务支持的搜索结果的组合。每个组件搜索的分数相加以获得结果的总分。

要执行混合搜索，可以将一个可选参数 `search_options` 传递给所有相似性搜索。  
有关 `search_options` 的不同搜索/查询可能性，请参见 [这里](https://docs.couchbase.com/server/current/search/search-request-params.html#query-object)。

### 创建多样化的混合搜索元数据
为了模拟混合搜索，让我们从现有文档中创建一些随机元数据。我们统一为元数据添加三个字段，`date` 在2010年到2020年之间，`rating` 在1到5之间，`author` 设置为 John Doe 或 Jane Doe。 

```python
# Adding metadata to documents
for i, doc in enumerate(docs):
    doc.metadata["date"] = f"{range(2010, 2020)[i % 10]}-01-01"
    doc.metadata["rating"] = range(1, 6)[i % 5]
    doc.metadata["author"] = ["John Doe", "Jane Doe"][i % 2]

vector_store.add_documents(docs)

query = "What did the president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(query)
print(results[0].metadata)
```
```output
{'author': 'John Doe', 'date': '2016-01-01', 'rating': 2, 'source': '../../how_to/state_of_the_union.txt'}
```

### 示例：按精确值搜索
我们可以在 `metadata` 对象中搜索文本字段的精确匹配，例如作者。

```python
query = "What did the president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(
    query,
    search_options={"query": {"field": "metadata.author", "match": "John Doe"}},
    fields=["metadata.author"],
)
print(results[0])
```
```output
page_content='This is personal to me and Jill, to Kamala, and to so many of you. \n\nCancer is the #2 cause of death in America–second only to heart disease. \n\nLast month, I announced our plan to supercharge  \nthe Cancer Moonshot that President Obama asked me to lead six years ago. \n\nOur goal is to cut the cancer death rate by at least 50% over the next 25 years, turn more cancers from death sentences into treatable diseases.  \n\nMore support for patients and families.' metadata={'author': 'John Doe'}
```

### 示例：按部分匹配搜索
我们可以通过指定模糊性来搜索部分匹配。这在您想要搜索搜索查询的轻微变体或拼写错误时非常有用。

在这里，“Jae”与“Jane”接近（模糊性为1）。

```python
query = "What did the president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(
    query,
    search_options={
        "query": {"field": "metadata.author", "match": "Jae", "fuzziness": 1}
    },
    fields=["metadata.author"],
)
print(results[0])
```
```output
page_content='A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. \n\nAnd if we are to advance liberty and justice, we need to secure the Border and fix the immigration system.' metadata={'author': 'Jane Doe'}
```

### 示例：按日期范围查询搜索
我们可以在日期字段如 `metadata.date` 上进行日期范围查询来搜索文档。

```python
query = "Any mention about independence?"
results = vector_store.similarity_search(
    query,
    search_options={
        "query": {
            "start": "2016-12-31",
            "end": "2017-01-02",
            "inclusive_start": True,
            "inclusive_end": False,
            "field": "metadata.date",
        }
    },
)
print(results[0])
```
```output
page_content='He will never extinguish their love of freedom. He will never weaken the resolve of the free world. \n\nWe meet tonight in an America that has lived through two of the hardest years this nation has ever faced. \n\nThe pandemic has been punishing. \n\nAnd so many families are living paycheck to paycheck, struggling to keep up with the rising cost of food, gas, housing, and so much more. \n\nI understand.' metadata={'author': 'Jane Doe', 'date': '2017-01-01', 'rating': 3, 'source': '../../how_to/state_of_the_union.txt'}
```

### 示例：按数值范围查询
我们可以搜索在数值字段如 `metadata.rating` 范围内的文档。

```python
query = "Any mention about independence?"
results = vector_store.similarity_search_with_score(
    query,
    search_options={
        "query": {
            "min": 3,
            "max": 5,
            "inclusive_min": True,
            "inclusive_max": True,
            "field": "metadata.rating",
        }
    },
)
print(results[0])
```
```output
(Document(page_content='He will never extinguish their love of freedom. He will never weaken the resolve of the free world. \n\nWe meet tonight in an America that has lived through two of the hardest years this nation has ever faced. \n\nThe pandemic has been punishing. \n\nAnd so many families are living paycheck to paycheck, struggling to keep up with the rising cost of food, gas, housing, and so much more. \n\nI understand.', metadata={'author': 'Jane Doe', 'date': '2017-01-01', 'rating': 3, 'source': '../../how_to/state_of_the_union.txt'}), 0.9000703597577832)
```

### 示例：组合多个搜索查询
不同的搜索查询可以使用 AND（合取）或 OR（析取）运算符进行组合。

在本示例中，我们正在检查评级在 3 和 4 之间，并且日期在 2015 年和 2018 年之间的文档。

```python
query = "Any mention about independence?"
results = vector_store.similarity_search_with_score(
    query,
    search_options={
        "query": {
            "conjuncts": [
                {"min": 3, "max": 4, "inclusive_max": True, "field": "metadata.rating"},
                {"start": "2016-12-31", "end": "2017-01-02", "field": "metadata.date"},
            ]
        }
    },
)
print(results[0])
```
```output
(Document(page_content='He will never extinguish their love of freedom. He will never weaken the resolve of the free world. \n\nWe meet tonight in an America that has lived through two of the hardest years this nation has ever faced. \n\nThe pandemic has been punishing. \n\nAnd so many families are living paycheck to paycheck, struggling to keep up with the rising cost of food, gas, housing, and so much more. \n\nI understand.', metadata={'author': 'Jane Doe', 'date': '2017-01-01', 'rating': 3, 'source': '../../how_to/state_of_the_union.txt'}), 1.3598770370389914)
```

### 其他查询
同样，您可以在 `search_options` 参数中使用任何支持的查询方法，如地理距离、区域搜索、通配符、正则表达式等。有关可用查询方法及其语法的更多详细信息，请参阅文档。

- [Couchbase Capella](https://docs.couchbase.com/cloud/search/search-request-params.html#query-object)
- [Couchbase Server](https://docs.couchbase.com/server/current/search/search-request-params.html#query-object)

# 常见问题解答

## 问题：我应该在创建 CouchbaseVectorStore 对象之前创建搜索索引吗？
是的，目前您需要在创建 `CouchbaseVectoreStore` 对象之前创建搜索索引。

## 问题：我在搜索结果中没有看到我指定的所有字段。

在 Couchbase 中，我们只能返回存储在搜索索引中的字段。请确保您尝试在搜索结果中访问的字段是搜索索引的一部分。

处理此问题的一种方法是动态地在索引中索引和存储文档的字段。

- 在 Capella 中，您需要进入“高级模式”，然后在“常规设置”下的下拉菜单中，您可以勾选“[X] 存储动态字段”或“[X] 索引动态字段”
- 在 Couchbase Server 中，在索引编辑器（而不是快速编辑器）下的下拉菜单“高级”中，您可以勾选“[X] 存储动态字段”或“[X] 索引动态字段”

请注意，这些选项会增加索引的大小。

有关动态映射的更多详细信息，请参阅 [文档](https://docs.couchbase.com/cloud/search/customize-index.html).

## 问题：我无法在搜索结果中看到元数据对象。
这很可能是由于文档中的 `metadata` 字段没有被 Couchbase Search 索引索引和/或存储。为了将文档中的 `metadata` 字段进行索引，您需要将其作为子映射添加到索引中。

如果您选择映射映射中的所有字段，您将能够通过所有元数据字段进行搜索。或者，为了优化索引，您可以选择 `metadata` 对象内的特定字段进行索引。您可以参考 [docs](https://docs.couchbase.com/cloud/search/customize-index.html) 了解更多关于索引子映射的信息。

创建子映射

* [Couchbase Capella](https://docs.couchbase.com/cloud/search/create-child-mapping.html)
* [Couchbase Server](https://docs.couchbase.com/server/current/search/create-child-mapping.html)

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)