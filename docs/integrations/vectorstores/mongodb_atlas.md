---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/mongodb_atlas.ipynb
---

# MongoDB Atlas

本笔记涵盖如何在 LangChain 中使用 `langchain-mongodb` 包进行 MongoDB Atlas 向量搜索。

>[MongoDB Atlas](https://www.mongodb.com/docs/atlas/) 是一个完全托管的云数据库，适用于 AWS、Azure 和 GCP。它支持对 MongoDB 文档数据进行原生向量搜索和全文搜索（BM25）。

>[MongoDB Atlas 向量搜索](https://www.mongodb.com/products/platform/atlas-vector-search) 允许将嵌入存储在 MongoDB 文档中，创建向量搜索索引，并使用近似最近邻算法（`Hierarchical Navigable Small Worlds`）执行 KNN 搜索。它使用 [$vectorSearch MQL Stage](https://www.mongodb.com/docs/atlas/atlas-vector-search/vector-search-overview/)。

## 前提条件
>*一个运行 MongoDB 版本 6.0.11、7.0.2 或更高版本（包括 RC）的 Atlas 集群。

>*一个 OpenAI API 密钥。您必须拥有一个具有可用于 API 请求的余额的付费 OpenAI 账户。

您需要安装 `langchain-mongodb` 才能使用此集成。

## 设置 MongoDB Atlas 集群
要使用 MongoDB Atlas，您必须首先部署一个集群。我们提供了一个永久免费使用的集群层级。要开始，请访问 Atlas：[快速入门](https://www.mongodb.com/docs/atlas/getting-started/)。

## 使用方法
在本笔记本中，我们将演示如何使用 MongoDB Atlas、OpenAI 和 Langchain 执行 `Retrieval Augmented Generation` (RAG)。我们将进行相似性搜索、带元数据预过滤的相似性搜索，以及对 2023 年 3 月发布的 [GPT 4 技术报告](https://arxiv.org/pdf/2303.08774.pdf) 的问答，该报告不在 OpenAI 的大型语言模型（LLM）的参数记忆中，因为其知识截止于 2021 年 9 月。

我们希望使用 `OpenAIEmbeddings`，因此需要设置我们的 OpenAI API 密钥。

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

现在我们将为 MongoDB Atlas 集群设置环境变量。

```python
%pip install --upgrade --quiet langchain langchain-mongodb pypdf pymongo langchain-openai tiktoken
```

```python
import getpass

MONGODB_ATLAS_CLUSTER_URI = getpass.getpass("MongoDB Atlas Cluster URI:")
```

```python
from pymongo import MongoClient

# initialize MongoDB python client
client = MongoClient(MONGODB_ATLAS_CLUSTER_URI)

DB_NAME = "langchain_db"
COLLECTION_NAME = "test"
ATLAS_VECTOR_SEARCH_INDEX_NAME = "index_name"

MONGODB_COLLECTION = client[DB_NAME][COLLECTION_NAME]
```

## 创建向量搜索索引

现在，让我们在您的集群上创建一个向量搜索索引。更详细的步骤可以在 [为 LangChain 创建向量搜索索引](https://www.mongodb.com/docs/atlas/atlas-vector-search/ai-integrations/langchain/#create-the-atlas-vector-search-index) 部分找到。在下面的示例中，`embedding` 是包含嵌入向量的字段名称。有关如何定义 Atlas 向量搜索索引的更多详细信息，请参阅 [文档](https://www.mongodb.com/docs/atlas/atlas-vector-search/create-index/)。
您可以将索引命名为 `{ATLAS_VECTOR_SEARCH_INDEX_NAME}`，并在命名空间 `{DB_NAME}.{COLLECTION_NAME}` 上创建该索引。最后，在 MongoDB Atlas 的 JSON 编辑器中写入以下定义：

```json
{
  "fields":[
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 1536,
      "similarity": "cosine"
    }
  ]
}
```

此外，如果您正在运行 MongoDB M10 集群，且服务器版本为 6.0 及以上，您可以利用 `MongoDBAtlasVectorSearch.create_index`。要添加上述索引，其用法如下所示。

```python
from langchain_community.embeddings.openai import OpenAIEmbeddings
from langchain_mongodb.vectorstores import MongoDBAtlasVectorSearch
from pymongo import MongoClient

mongo_client = MongoClient("<YOUR-CONNECTION-STRING>")
collection = mongo_client["<db_name>"]["<collection_name>"]
embeddings = OpenAIEmbeddings()

vectorstore = MongoDBAtlasVectorSearch(
  collection=collection,
  embedding=embeddings,
  index_name="<ATLAS_VECTOR_SEARCH_INDEX_NAME>",
  relevance_score_fn="cosine",
)

# 使用提供的 index_name 和 relevance_score_fn 类型创建索引
vectorstore.create_index(dimensions=1536)
```

# 插入数据


```python
from langchain_community.document_loaders import PyPDFLoader

# Load the PDF
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


```python
from langchain_community.vectorstores import MongoDBAtlasVectorSearch
from langchain_openai import OpenAIEmbeddings

# insert the documents in MongoDB Atlas with their embedding
vector_search = MongoDBAtlasVectorSearch.from_documents(
    documents=docs,
    embedding=OpenAIEmbeddings(disallowed_special=()),
    collection=MONGODB_COLLECTION,
    index_name=ATLAS_VECTOR_SEARCH_INDEX_NAME,
)
```


```python
# Perform a similarity search between the embedding of the query and the embeddings of the documents
query = "What were the compute requirements for training GPT 4"
results = vector_search.similarity_search(query)

print(results[0].page_content)
```

# 查询数据

我们还可以直接实例化向量存储并执行查询，如下所示：

```python
from langchain_community.vectorstores import MongoDBAtlasVectorSearch
from langchain_openai import OpenAIEmbeddings

vector_search = MongoDBAtlasVectorSearch.from_connection_string(
    MONGODB_ATLAS_CLUSTER_URI,
    DB_NAME + "." + COLLECTION_NAME,
    OpenAIEmbeddings(disallowed_special=()),
    index_name=ATLAS_VECTOR_SEARCH_INDEX_NAME,
)
```

## 使用相似性搜索进行预过滤

Atlas Vector Search 支持使用 MQL 操作符进行过滤的预过滤。以下是一个示例索引和查询，基于上述加载的相同数据，允许您对“page”字段进行元数据过滤。您可以使用定义的过滤器更新现有索引，并进行向量搜索的预过滤。

```json
{
  "fields":[
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 1536,
      "similarity": "cosine"
    },
    {
      "type": "filter",
      "path": "page"
    }
  ]
}
```

您还可以使用 `MongoDBAtlasVectorSearch.create_index` 方法以编程方式更新索引。

```python
vectorstore.create_index(
  dimensions=1536,
  filters=[{"type":"filter", "path":"page"}],
  update=True
)
```


```python
query = "What were the compute requirements for training GPT 4"

results = vector_search.similarity_search_with_score(
    query=query, k=5, pre_filter={"page": {"$eq": 1}}
)

# Display results
for result in results:
    print(result)
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

## 问答系统

```python
qa_retriever = vector_search.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 25},
)
```

```python
from langchain_core.prompts import PromptTemplate

prompt_template = """使用以下上下文片段回答最后的问题。如果你不知道答案，就说你不知道，不要试图编造答案。

{context}

问题: {question}
"""
PROMPT = PromptTemplate(
    template=prompt_template, input_variables=["context", "question"]
)
```

```python
from langchain.chains import RetrievalQA
from langchain_openai import OpenAI

qa = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",
    retriever=qa_retriever,
    return_source_documents=True,
    chain_type_kwargs={"prompt": PROMPT},
)

docs = qa({"query": "gpt-4 计算需求"})

print(docs["result"])
print(docs["source_documents"])
```

GPT-4 需要的计算量显著高于早期的 GPT 模型。在一个源自 OpenAI 内部代码库的数据集中，GPT-4 需要 100p（千万亿次浮点运算）的计算才能达到最低损失，而较小的模型只需 1-10n（十亿次浮点运算）。

# 其他说明
>* 更多文档可以在 [LangChain-MongoDB](https://www.mongodb.com/docs/atlas/atlas-vector-search/ai-integrations/langchain/) 网站上找到
>* 此功能已普遍可用，准备进行生产部署。
>* langchain 版本 0.0.305 ([发布说明](https://github.com/langchain-ai/langchain/releases/tag/v0.0.305)) 引入了对 $vectorSearch MQL 阶段的支持，该功能在 MongoDB Atlas 6.0.11 和 7.0.2 中可用。使用早期版本 MongoDB Atlas 的用户需要将其 LangChain 版本固定为 <=0.0.304
>

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)