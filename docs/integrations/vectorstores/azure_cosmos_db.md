---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/azure_cosmos_db.ipynb
---

# Azure Cosmos DB Mongo vCore

本笔记本展示了如何利用这个集成的 [向量数据库](https://learn.microsoft.com/en-us/azure/cosmos-db/vector-database) 在集合中存储文档、创建索引，并使用近似最近邻算法（如 COS（余弦距离）、L2（欧几里得距离）和 IP（内积））执行向量搜索查询，以定位与查询向量接近的文档。

Azure Cosmos DB 是支持 OpenAI 的 ChatGPT 服务的数据库。它提供单毫秒级的响应时间、自动和即时的可扩展性，并在任何规模下保证速度。

Azure Cosmos DB for MongoDB vCore(https://learn.microsoft.com/en-us/azure/cosmos-db/mongodb/vcore/) 为开发人员提供了一个完全托管的 MongoDB 兼容数据库服务，以构建具有熟悉架构的现代应用程序。您可以利用您的 MongoDB 经验，继续使用您喜欢的 MongoDB 驱动程序、SDK 和工具，只需将您的应用程序指向 MongoDB vCore 账户的连接字符串的 API。

[注册](https://azure.microsoft.com/en-us/free/) 获取终身免费访问权限，今天就开始吧。

```python
%pip install --upgrade --quiet  pymongo langchain-openai langchain-community
```
```output

[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m A new release of pip is available: [0m[31;49m23.2.1[0m[39;49m -> [0m[32;49m23.3.2[0m
[1m[[0m[34;49mnotice[0m[1;39;49m][0m[39;49m To update, run: [0m[32;49mpip install --upgrade pip[0m
Note: you may need to restart the kernel to use updated packages.
```

```python
import os

CONNECTION_STRING = "YOUR_CONNECTION_STRING"
INDEX_NAME = "izzy-test-index"
NAMESPACE = "izzy_test_db.izzy_test_collection"
DB_NAME, COLLECTION_NAME = NAMESPACE.split(".")
```

我们想使用 `OpenAIEmbeddings`，因此需要设置我们的 Azure OpenAI API 密钥以及其他环境变量。

```python
# 设置 OpenAI 环境变量
os.environ["OPENAI_API_TYPE"] = "azure"
os.environ["OPENAI_API_VERSION"] = "2023-05-15"
os.environ["OPENAI_API_BASE"] = (
    "YOUR_OPEN_AI_ENDPOINT"  # https://example.openai.azure.com/
)
os.environ["OPENAI_API_KEY"] = "YOUR_OPENAI_API_KEY"
os.environ["OPENAI_EMBEDDINGS_DEPLOYMENT"] = (
    "smart-agent-embedding-ada"  # 嵌入模型的部署名称
)
os.environ["OPENAI_EMBEDDINGS_MODEL_NAME"] = "text-embedding-ada-002"  # 模型名称
```

现在，我们需要将文档加载到集合中，创建索引，然后对索引执行查询以检索匹配项。

如果您对某些参数有疑问，请参考 [文档](https://learn.microsoft.com/en-us/azure/cosmos-db/mongodb/vcore/vector-search)。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.azure_cosmos_db import (
    AzureCosmosDBVectorSearch,
    CosmosDBSimilarityType,
    CosmosDBVectorSearchType,
)
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

SOURCE_FILE_NAME = "../../how_to/state_of_the_union.txt"

loader = TextLoader(SOURCE_FILE_NAME)
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

# OpenAI 设置
model_deployment = os.getenv(
    "OPENAI_EMBEDDINGS_DEPLOYMENT", "smart-agent-embedding-ada"
)
model_name = os.getenv("OPENAI_EMBEDDINGS_MODEL_NAME", "text-embedding-ada-002")


openai_embeddings: OpenAIEmbeddings = OpenAIEmbeddings(
    deployment=model_deployment, model=model_name, chunk_size=1
)
```

```python
from pymongo import MongoClient

# INDEX_NAME = "izzy-test-index-2"
# NAMESPACE = "izzy_test_db.izzy_test_collection"
# DB_NAME, COLLECTION_NAME = NAMESPACE.split(".")

client: MongoClient = MongoClient(CONNECTION_STRING)
collection = client[DB_NAME][COLLECTION_NAME]

model_deployment = os.getenv(
    "OPENAI_EMBEDDINGS_DEPLOYMENT", "smart-agent-embedding-ada"
)
model_name = os.getenv("OPENAI_EMBEDDINGS_MODEL_NAME", "text-embedding-ada-002")

vectorstore = AzureCosmosDBVectorSearch.from_documents(
    docs,
    openai_embeddings,
    collection=collection,
    index_name=INDEX_NAME,
)

# 在此处详细了解这些变量。 https://learn.microsoft.com/en-us/azure/cosmos-db/mongodb/vcore/vector-search
num_lists = 100
dimensions = 1536
similarity_algorithm = CosmosDBSimilarityType.COS
kind = CosmosDBVectorSearchType.VECTOR_IVF
m = 16
ef_construction = 64
ef_search = 40
score_threshold = 0.1

vectorstore.create_index(
    num_lists, dimensions, similarity_algorithm, kind, m, ef_construction
)
```

```output
{'raw': {'defaultShard': {'numIndexesBefore': 1,
   'numIndexesAfter': 2,
   'createdCollectionAutomatically': False,
   'ok': 1}},
 'ok': 1}
```

```python
# 在查询的嵌入与文档的嵌入之间执行相似性搜索
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = vectorstore.similarity_search(query)
```

```python
print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯投票权法案》。同时，推动《披露法案》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：史蒂芬·布雷耶大法官——一位退伍军人、宪法学者以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最重要的宪法责任之一是提名某人担任美国最高法院法官。

四天前，我提名了巡回上诉法庭法官 Ketanji Brown Jackson。她是我们国家顶尖的法律人才，将延续布雷耶大法官卓越的遗产。
```
一旦文档加载完毕并创建了索引，您现在可以直接实例化向量存储并对索引运行查询。

```python
vectorstore = AzureCosmosDBVectorSearch.from_connection_string(
    CONNECTION_STRING, NAMESPACE, openai_embeddings, index_name=INDEX_NAME
)

# 在查询与已摄取文档之间执行相似性搜索
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = vectorstore.similarity_search(query)

print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯投票权法案》。同时，推动《披露法案》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：史蒂芬·布雷耶大法官——一位退伍军人、宪法学者以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最重要的宪法责任之一是提名某人担任美国最高法院法官。

四天前，我提名了巡回上诉法庭法官 Ketanji Brown Jackson。她是我们国家顶尖的法律人才，将延续布雷耶大法官卓越的遗产。
```

```python
vectorstore = AzureCosmosDBVectorSearch(
    collection, openai_embeddings, index_name=INDEX_NAME
)

# 在查询与已摄取文档之间执行相似性搜索
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = vectorstore.similarity_search(query)

print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯投票权法案》。同时，推动《披露法案》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：史蒂芬·布雷耶大法官——一位退伍军人、宪法学者以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最重要的宪法责任之一是提名某人担任美国最高法院法官。

四天前，我提名了巡回上诉法庭法官 Ketanji Brown Jackson。她是我们国家顶尖的法律人才，将延续布雷耶大法官卓越的遗产。
```

## 过滤后的向量搜索（预览）
Azure Cosmos DB for MongoDB 支持使用 $lt, $lte, $eq, $neq, $gte, $gt, $in, $nin 和 $regex 进行预过滤。要使用此功能，请在您的 Azure 订阅的“预览功能”选项卡中启用“过滤向量搜索”。有关预览功能的更多信息，请[点击这里](https://learn.microsoft.com/azure/cosmos-db/mongodb/vcore/vector-search#filtered-vector-search-preview).

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)