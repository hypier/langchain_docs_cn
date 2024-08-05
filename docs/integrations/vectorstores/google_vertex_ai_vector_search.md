---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/google_vertex_ai_vector_search.ipynb
---

# Google Vertex AI 向量搜索

本笔记本展示了如何使用与 `Google Cloud Vertex AI 向量搜索` 向量数据库相关的功能。

> [Google Vertex AI 向量搜索](https://cloud.google.com/vertex-ai/docs/vector-search/overview)，前称为 Vertex AI 匹配引擎，提供行业领先的高规模低延迟向量数据库。这些向量数据库通常被称为向量相似性匹配或近似最近邻（ANN）服务。

**注意**：Langchain API 期望已经创建的端点和部署的索引。索引创建时间可能需要长达一小时。

> 要了解如何创建索引，请参考部分 [创建索引并将其部署到端点](#create-index-and-deploy-it-to-an-endpoint)  
如果您已经部署了索引，请跳至 [从文本创建向量存储](#create-vector-store-from-texts)

## 创建索引并将其部署到端点
- 本节演示如何创建新索引并将其部署到端点


```python
# TODO : Set values as per your requirements
# Project and Storage Constants
PROJECT_ID = "<my_project_id>"
REGION = "<my_region>"
BUCKET = "<my_gcs_bucket>"
BUCKET_URI = f"gs://{BUCKET}"

# The number of dimensions for the textembedding-gecko@003 is 768
# If other embedder is used, the dimensions would probably need to change.
DIMENSIONS = 768

# Index Constants
DISPLAY_NAME = "<my_matching_engine_index_id>"
DEPLOYED_INDEX_ID = "<my_matching_engine_endpoint_id>"
```


```python
# Create a bucket.
! gsutil mb -l $REGION -p $PROJECT_ID $BUCKET_URI
```

### 使用 [VertexAIEmbeddings](https://python.langchain.com/docs/integrations/text_embedding/google_vertex_ai_palm/) 作为嵌入模型


```python
from google.cloud import aiplatform
from langchain_google_vertexai import VertexAIEmbeddings
```


```python
aiplatform.init(project=PROJECT_ID, location=REGION, staging_bucket=BUCKET_URI)
```


```python
embedding_model = VertexAIEmbeddings(model_name="textembedding-gecko@003")
```

### 创建空索引 

**注意：** 创建索引时，您应该从“BATCH_UPDATE”或“STREAM_UPDATE”中指定一个“index_update_method”
> 批量索引适用于您希望在一段时间内存储的数据上批量更新索引的情况，例如每周或每月处理的系统。流索引则是在您希望随着新数据添加到数据存储中而更新索引数据的情况下，例如，如果您有一家书店，并希望尽快在线展示新库存。您选择哪种类型很重要，因为设置和要求是不同的。

有关配置索引的更多详细信息，请参阅 [官方文档](https://cloud.google.com/vertex-ai/docs/vector-search/create-manage-index#create-index-batch)



```python
# NOTE : This operation can take upto 30 seconds
my_index = aiplatform.MatchingEngineIndex.create_tree_ah_index(
    display_name=DISPLAY_NAME,
    dimensions=DIMENSIONS,
    approximate_neighbors_count=150,
    distance_measure_type="DOT_PRODUCT_DISTANCE",
    index_update_method="STREAM_UPDATE",  # allowed values BATCH_UPDATE , STREAM_UPDATE
)
```

### 创建一个端点


```python
# Create an endpoint
my_index_endpoint = aiplatform.MatchingEngineIndexEndpoint.create(
    display_name=f"{DISPLAY_NAME}-endpoint", public_endpoint_enabled=True
)
```

### 将索引部署到端点


```python
# NOTE : This operation can take upto 20 minutes
my_index_endpoint = my_index_endpoint.deploy_index(
    index=my_index, deployed_index_id=DEPLOYED_INDEX_ID
)

my_index_endpoint.deployed_indexes
```

## 从文本创建向量存储

注意：如果您已有现有的索引和端点，可以使用以下代码加载它们。

```python
# TODO : replace 1234567890123456789 with your acutial index ID
my_index = aiplatform.MatchingEngineIndex("1234567890123456789")

# TODO : replace 1234567890123456789 with your acutial endpoint ID
my_index_endpoint = aiplatform.MatchingEngineIndexEndpoint("1234567890123456789")
```

```python
from langchain_google_vertexai import (
    VectorSearchVectorStore,
    VectorSearchVectorStoreDatastore,
)
```

### 创建简单的向量存储（无过滤器）

```python
# Input texts
texts = [
    "The cat sat on",
    "the mat.",
    "I like to",
    "eat pizza for",
    "dinner.",
    "The sun sets",
    "in the west.",
]

# Create a Vector Store
vector_store = VectorSearchVectorStore.from_components(
    project_id=PROJECT_ID,
    region=REGION,
    gcs_bucket_name=BUCKET,
    index_id=my_index.name,
    endpoint_id=my_index_endpoint.name,
    embedding=embedding_model,
    stream_update=True,
)

# Add vectors and mapped text chunks to your vectore store
vector_store.add_texts(texts=texts)
```

### 可选：您还可以创建向量并将块存储在数据存储中

```python
# NOTE : This operation can take upto 20 mins
vector_store = VectorSearchVectorStoreDatastore.from_components(
    project_id=PROJECT_ID,
    region=REGION,
    index_id=my_index.name,
    endpoint_id=my_index_endpoint.name,
    embedding=embedding_model,
    stream_update=True,
)

vector_store.add_texts(texts=texts, is_complete_overwrite=True)
```

```python
# Try running a simialarity search
vector_store.similarity_search("pizza")
```

### 创建带有元数据过滤器的向量存储


```python
# 输入文本与元数据
record_data = [
    {
        "description": "一条多功能的深色牛仔裤。"
        "采用耐用的棉料，经典的直筒剪裁，这条牛仔裤"
        "能轻松从休闲日转变为更正式的场合。",
        "price": 65.00,
        "color": "blue",
        "season": ["fall", "winter", "spring"],
    },
    {
        "description": "一件轻便的亚麻扣领衬衫，洁白如新。"
        "非常适合在透气面料和宽松剪裁下保持凉爽。",
        "price": 34.99,
        "color": "white",
        "season": ["summer", "spring"],
    },
    {
        "description": "一件柔软的粗针织毛衣，生机勃勃的森林绿色。"
        "超大号的剪裁和舒适的羊毛混纺使其非常适合在温度下降时保持温暖。",
        "price": 89.99,
        "color": "green",
        "season": ["fall", "winter"],
    },
    {
        "description": "一件经典的圆领T恤，柔软的混色蓝色。"
        "由舒适的棉质平纹布制成，这件T恤是每个季节的衣橱必备单品。",
        "price": 19.99,
        "color": "blue",
        "season": ["fall", "winter", "summer", "spring"],
    },
    {
        "description": "一条流畅的中长裙，带有精致的花卉图案。"
        "轻盈通透，这条裙子为温暖的日子增添了一丝女性风格。",
        "price": 45.00,
        "color": "white",
        "season": ["spring", "summer"],
    },
]
```


```python
# 解析并准备输入数据

texts = []
metadatas = []
for record in record_data:
    record = record.copy()
    page_content = record.pop("description")
    texts.append(page_content)
    if isinstance(page_content, str):
        metadata = {**record}
        metadatas.append(metadata)
```


```python
# 检查元数据
metadatas
```


```python
# 注意：此操作可能需要超过20分钟
vector_store = VectorSearchVectorStore.from_components(
    project_id=PROJECT_ID,
    region=REGION,
    gcs_bucket_name=BUCKET,
    index_id=my_index.name,
    endpoint_id=my_index_endpoint.name,
    embedding=embedding_model,
)

vector_store.add_texts(texts=texts, metadatas=metadatas, is_complete_overwrite=True)
```


```python
from google.cloud.aiplatform.matching_engine.matching_engine_index_endpoint import (
    Namespace,
    NumericNamespace,
)
```


```python
# 尝试运行简单的相似性搜索

# 以下代码应返回5个结果
vector_store.similarity_search("shirt", k=5)
```


```python
# 尝试运行带有文本过滤器的相似性搜索
filters = [Namespace(name="season", allow_tokens=["spring"])]

# 以下代码现在应返回4个结果
vector_store.similarity_search("shirt", k=5, filter=filters)
```


```python
# 尝试运行结合文本和数值过滤器的相似性搜索
filters = [Namespace(name="season", allow_tokens=["spring"])]
numeric_filters = [NumericNamespace(name="price", value_float=40.0, op="LESS")]

# 以下代码现在应返回2个结果
vector_store.similarity_search(
    "shirt", k=5, filter=filters, numeric_filter=numeric_filters
)
```

### 使用向量存储作为检索器


```python
# Initialize the vectore_store as retriever
retriever = vector_store.as_retriever()
```


```python
# perform simple similarity search on retriever
retriever.invoke("What are my options in breathable fabric?")
```


```python
# Try running a similarity search with text filter
filters = [Namespace(name="season", allow_tokens=["spring"])]

retriever.search_kwargs = {"filter": filters}

# perform similarity search with filters on retriever
retriever.invoke("What are my options in breathable fabric?")
```


```python
# Try running a similarity search with combination of text and numeric filter
filters = [Namespace(name="season", allow_tokens=["spring"])]
numeric_filters = [NumericNamespace(name="price", value_float=40.0, op="LESS")]


retriever.search_kwargs = {"filter": filters, "numeric_filter": numeric_filters}

retriever.invoke("What are my options in breathable fabric?")
```

### 在问答链中使用检索器的过滤器


```python
from langchain_google_vertexai import VertexAI

llm = VertexAI(model_name="gemini-pro")
```


```python
from langchain.chains import RetrievalQA

filters = [Namespace(name="season", allow_tokens=["spring"])]
numeric_filters = [NumericNamespace(name="price", value_float=40.0, op="LESS")]

retriever.search_kwargs = {"k": 2, "filter": filters, "numeric_filter": numeric_filters}

retrieval_qa = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=retriever,
    return_source_documents=True,
)

question = "What are my options in breathable fabric?"
response = retrieval_qa({"query": question})
print(f"{response['result']}")
print("REFERENCES")
print(f"{response['source_documents']}")
```

## 读取、分块、向量化和索引 PDFs


```python
!pip install pypdf
```


```python
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
```


```python
loader = PyPDFLoader("https://arxiv.org/pdf/1706.03762.pdf")
pages = loader.load()
```


```python
text_splitter = RecursiveCharacterTextSplitter(
    # 设置一个非常小的块大小，仅用于演示。
    chunk_size=1000,
    chunk_overlap=20,
    length_function=len,
    is_separator_regex=False,
)
doc_splits = text_splitter.split_documents(pages)
```


```python
texts = [doc.page_content for doc in doc_splits]
metadatas = [doc.metadata for doc in doc_splits]
```


```python
texts[0]
```


```python
# 检查第一页的元数据
metadatas[0]
```


```python
vector_store = VectorSearchVectorStore.from_components(
    project_id=PROJECT_ID,
    region=REGION,
    gcs_bucket_name=BUCKET,
    index_id=my_index.name,
    endpoint_id=my_index_endpoint.name,
    embedding=embedding_model,
)

vector_store.add_texts(texts=texts, metadatas=metadatas, is_complete_overwrite=True)
```


```python
vector_store = VectorSearchVectorStore.from_components(
    project_id=PROJECT_ID,
    region=REGION,
    gcs_bucket_name=BUCKET,
    index_id=my_index.name,
    endpoint_id=my_index_endpoint.name,
    embedding=embedding_model,
)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)