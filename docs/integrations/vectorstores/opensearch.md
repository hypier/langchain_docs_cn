---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/opensearch.ipynb
---

# OpenSearch

> [OpenSearch](https://opensearch.org/) 是一个可扩展、灵活且可扩展的开源软件套件，用于搜索、分析和可观察性应用，采用 Apache 2.0 许可证。 `OpenSearch` 是一个基于 `Apache Lucene` 的分布式搜索和分析引擎。

本笔记本展示了如何使用与 `OpenSearch` 数据库相关的功能。

要运行，您需要有一个正在运行的 OpenSearch 实例：[请查看这里以获取简单的 Docker 安装](https://hub.docker.com/r/opensearchproject/opensearch)。

`similarity_search` 默认执行近似 k-NN 搜索，使用多种算法之一，如 lucene、nmslib、faiss，推荐用于大数据集。要执行暴力搜索，我们还有其他被称为脚本评分和 Painless 脚本的搜索方法。
有关更多详细信息，请查看 [此处](https://opensearch.org/docs/latest/search-plugins/knn/index/)。

## 安装
安装 Python 客户端。


```python
%pip install --upgrade --quiet  opensearch-py langchain-community
```

我们想要使用 OpenAIEmbeddings，因此我们需要获取 OpenAI API 密钥。


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```


```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import OpenSearchVectorSearch
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```


```python
from langchain_community.document_loaders import TextLoader

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```

## 使用近似 k-NN 的 similarity_search

使用自定义参数的 `similarity_search` 进行 `Approximate k-NN` 搜索


```python
docsearch = OpenSearchVectorSearch.from_documents(
    docs, embeddings, opensearch_url="http://localhost:9200"
)

# 如果使用默认的 Docker 安装，请使用此实例化：
# docsearch = OpenSearchVectorSearch.from_documents(
#     docs,
#     embeddings,
#     opensearch_url="https://localhost:9200",
#     http_auth=("admin", "admin"),
#     use_ssl = False,
#     verify_certs = False,
#     ssl_assert_hostname = False,
#     ssl_show_warn = False,
# )
```


```python
query = "What did the president say about Ketanji Brown Jackson"
docs = docsearch.similarity_search(query, k=10)
```


```python
print(docs[0].page_content)
```


```python
docsearch = OpenSearchVectorSearch.from_documents(
    docs,
    embeddings,
    opensearch_url="http://localhost:9200",
    engine="faiss",
    space_type="innerproduct",
    ef_construction=256,
    m=48,
)

query = "What did the president say about Ketanji Brown Jackson"
docs = docsearch.similarity_search(query)
```


```python
print(docs[0].page_content)
```

## 使用脚本评分的相似性搜索

使用自定义参数的 `similarity_search` 和 `Script Scoring`


```python
docsearch = OpenSearchVectorSearch.from_documents(
    docs, embeddings, opensearch_url="http://localhost:9200", is_appx_search=False
)

query = "What did the president say about Ketanji Brown Jackson"
docs = docsearch.similarity_search(
    "What did the president say about Ketanji Brown Jackson",
    k=1,
    search_type="script_scoring",
)
```


```python
print(docs[0].page_content)
```

## 使用 Painless 脚本的 similarity_search

使用带有自定义参数的 `similarity_search` 和 `Painless Scripting`


```python
docsearch = OpenSearchVectorSearch.from_documents(
    docs, embeddings, opensearch_url="http://localhost:9200", is_appx_search=False
)
filter = {"bool": {"filter": {"term": {"text": "smuggling"}}}}
query = "What did the president say about Ketanji Brown Jackson"
docs = docsearch.similarity_search(
    "What did the president say about Ketanji Brown Jackson",
    search_type="painless_scripting",
    space_type="cosineSimilarity",
    pre_filter=filter,
)
```


```python
print(docs[0].page_content)
```

## 最大边际相关性搜索 (MMR)
如果您想查找一些相似的文档，但又希望获得多样化的结果，MMR 是您应该考虑的方法。最大边际相关性在优化与查询的相似性和所选文档之间的多样性方面表现出色。

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = docsearch.max_marginal_relevance_search(query, k=2, fetch_k=10, lambda_param=0.5)
```

## 使用现有的 OpenSearch 实例

也可以使用已经存在的 OpenSearch 实例，该实例中的文档已经包含向量。

```python
# this is just an example, you would need to change these values to point to another opensearch instance
docsearch = OpenSearchVectorSearch(
    index_name="index-*",
    embedding_function=embeddings,
    opensearch_url="http://localhost:9200",
)

# you can specify custom field names to match the fields you're using to store your embedding, document text value, and metadata
docs = docsearch.similarity_search(
    "Who was asking about getting lunch today?",
    search_type="script_scoring",
    space_type="cosinesimil",
    vector_field="message_embedding",
    text_field="message",
    metadata_field="message_metadata",
)
```

## 使用 AOSS (Amazon OpenSearch Service Serverless)

这是一个使用 `faiss` 引擎和 `efficient_filter` 的 `AOSS` 示例。

我们需要安装几个 `python` 包。

```python
%pip install --upgrade --quiet  boto3 requests requests-aws4auth
```

```python
import boto3
from opensearchpy import RequestsHttpConnection
from requests_aws4auth import AWS4Auth

service = "aoss"  # must set the service as 'aoss'
region = "us-east-2"
credentials = boto3.Session(
    aws_access_key_id="xxxxxx", aws_secret_access_key="xxxxx"
).get_credentials()
awsauth = AWS4Auth("xxxxx", "xxxxxx", region, service, session_token=credentials.token)

docsearch = OpenSearchVectorSearch.from_documents(
    docs,
    embeddings,
    opensearch_url="host url",
    http_auth=awsauth,
    timeout=300,
    use_ssl=True,
    verify_certs=True,
    connection_class=RequestsHttpConnection,
    index_name="test-index-using-aoss",
    engine="faiss",
)

docs = docsearch.similarity_search(
    "What is feature selection",
    efficient_filter=filter,
    k=200,
)
```

## 使用 AOS (Amazon OpenSearch Service)


```python
%pip install --upgrade --quiet  boto3
```


```python
# This is just an example to show how to use Amazon OpenSearch Service, you need to set proper values.
import boto3
from opensearchpy import RequestsHttpConnection

service = "es"  # must set the service as 'es'
region = "us-east-2"
credentials = boto3.Session(
    aws_access_key_id="xxxxxx", aws_secret_access_key="xxxxx"
).get_credentials()
awsauth = AWS4Auth("xxxxx", "xxxxxx", region, service, session_token=credentials.token)

docsearch = OpenSearchVectorSearch.from_documents(
    docs,
    embeddings,
    opensearch_url="host url",
    http_auth=awsauth,
    timeout=300,
    use_ssl=True,
    verify_certs=True,
    connection_class=RequestsHttpConnection,
    index_name="test-index",
)

docs = docsearch.similarity_search(
    "What is feature selection",
    k=200,
)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)