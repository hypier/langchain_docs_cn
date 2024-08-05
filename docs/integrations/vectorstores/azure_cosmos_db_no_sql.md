---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/azure_cosmos_db_no_sql.ipynb
---

# Azure Cosmos DB No SQL

本笔记本展示了如何利用这个集成的[向量数据库](https://learn.microsoft.com/en-us/azure/cosmos-db/vector-database)在集合中存储文档，创建索引，并使用近似最近邻算法（如COS（余弦距离）、L2（欧几里得距离）和IP（内积））执行向量搜索查询，以定位与查询向量接近的文档。

Azure Cosmos DB是支持OpenAI的ChatGPT服务的数据库。它提供单毫秒级的响应时间，自动和即时的可扩展性，以及在任何规模下的速度保证。

[Azure Cosmos DB for NoSQL](https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/vector-search)现在提供向量索引和搜索的预览功能。此功能旨在处理高维向量，使得在任何规模下都能高效且准确地进行向量搜索。您现在可以将向量直接存储在文档中，与您的数据一起。这意味着数据库中的每个文档不仅可以包含传统的无模式数据，还可以包含作为文档其他属性的高维向量。数据和向量的共存使得索引和搜索更加高效，因为向量与它们所代表的数据存储在同一逻辑单元中。这简化了数据管理、AI应用架构和基于向量操作的效率。

[注册](https://azure.microsoft.com/en-us/free/)以获得终身免费访问权限，今天就开始吧。

```python
%pip install --upgrade --quiet azure-cosmos langchain-openai langchain-community
```
```output
Note: you may need to restart the kernel to use updated packages.
```

```python
OPENAI_API_KEY = "YOUR_KEY"
OPENAI_API_TYPE = "azure"
OPENAI_API_VERSION = "2023-05-15"
OPENAI_API_BASE = "YOUR_ENDPOINT"
OPENAI_EMBEDDINGS_MODEL_NAME = "text-embedding-ada-002"
OPENAI_EMBEDDINGS_MODEL_DEPLOYMENT = "text-embedding-ada-002"
```

## 插入数据


```python
from langchain_community.document_loaders import PyPDFLoader

# 加载PDF
loader = PyPDFLoader("https://arxiv.org/pdf/2303.08774.pdf")
data = loader.load()
```


```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=150)
docs = text_splitter.split_documents(data)
```


```python
print(docs[0])
```
```output
page_content='GPT-4 Technical Report\nOpenAI∗\nAbstract\nWe report the development of GPT-4, a large-scale, multimodal model which can\naccept image and text inputs and produce text outputs. While less capable than\nhumans in many real-world scenarios, GPT-4 exhibits human-level performance\non various professional and academic benchmarks, including passing a simulated\nbar exam with a score around the top 10% of test takers. GPT-4 is a Transformer-\nbased model pre-trained to predict the next token in a document. The post-training\nalignment process results in improved performance on measures of factuality and\nadherence to desired behavior. A core component of this project was developing\ninfrastructure and optimization methods that behave predictably across a wide\nrange of scales. This allowed us to accurately predict some aspects of GPT-4’s\nperformance based on models trained with no more than 1/1,000th the compute of\nGPT-4.\n1 Introduction' metadata={'source': 'https://arxiv.org/pdf/2303.08774.pdf', 'page': 0}
```

## 创建 AzureCosmosDB NoSQL 向量搜索


```python
indexing_policy = {
    "indexingMode": "consistent",
    "includedPaths": [{"path": "/*"}],
    "excludedPaths": [{"path": '/"_etag"/?'}],
    "vectorIndexes": [{"path": "/embedding", "type": "quantizedFlat"}],
}

vector_embedding_policy = {
    "vectorEmbeddings": [
        {
            "path": "/embedding",
            "dataType": "float32",
            "distanceFunction": "cosine",
            "dimensions": 1536,
        }
    ]
}
```


```python
from azure.cosmos import CosmosClient, PartitionKey
from langchain_community.vectorstores.azure_cosmos_db_no_sql import (
    AzureCosmosDBNoSqlVectorSearch,
)
from langchain_openai import AzureOpenAIEmbeddings

HOST = "AZURE_COSMOS_DB_ENDPOINT"
KEY = "AZURE_COSMOS_DB_KEY"

cosmos_client = CosmosClient(HOST, KEY)
database_name = "langchain_python_db"
container_name = "langchain_python_container"
partition_key = PartitionKey(path="/id")
cosmos_container_properties = {"partition_key": partition_key}

openai_embeddings = AzureOpenAIEmbeddings(
    azure_deployment=OPENAI_EMBEDDINGS_MODEL_DEPLOYMENT,
    api_version=OPENAI_API_VERSION,
    azure_endpoint=OPENAI_API_BASE,
    openai_api_key=OPENAI_API_KEY,
)

# 将文档及其嵌入插入 AzureCosmosDBNoSql
vector_search = AzureCosmosDBNoSqlVectorSearch.from_documents(
    documents=docs,
    embedding=openai_embeddings,
    cosmos_client=cosmos_client,
    database_name=database_name,
    container_name=container_name,
    vector_embedding_policy=vector_embedding_policy,
    indexing_policy=indexing_policy,
    cosmos_container_properties=cosmos_container_properties,
)
```

## 查询数据


```python
# 在查询的嵌入与文档的嵌入之间执行相似性搜索
query = "训练 GPT 4 的计算要求是什么"
results = vector_search.similarity_search(query)

print(results[0].page_content)
```
```output
performance based on models trained with no more than 1/1,000th the compute of
GPT-4.
1 Introduction
This technical report presents GPT-4, a large multimodal model capable of processing image and
text inputs and producing text outputs. Such models are an important area of study as they have the
potential to be used in a wide range of applications, such as dialogue systems, text summarization,
and machine translation. As such, they have been the subject of substantial interest and progress in
recent years [1–34].
One of the main goals of developing such models is to improve their ability to understand and generate
natural language text, particularly in more complex and nuanced scenarios. To test its capabilities
in such scenarios, GPT-4 was evaluated on a variety of exams originally designed for humans. In
these evaluations it performs quite well and often outscores the vast majority of human test takers.
```

## 相似性搜索与评分


```python
query = "What were the compute requirements for training GPT 4"

results = vector_search.similarity_search_with_score(
    query=query,
    k=5,
)

# Display results
for result in results:
    print(result)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)