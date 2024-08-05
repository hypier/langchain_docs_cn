---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/vdms.ipynb
---

# 英特尔的视觉数据管理系统 (VDMS)

> 英特尔的 [VDMS](https://github.com/IntelLabs/vdms) 是一个用于高效访问大“视觉”数据的存储解决方案，旨在通过搜索存储为图的相关视觉数据的视觉元数据来实现云规模，并为视觉数据启用机器友好的增强，以便更快地访问。VDMS 采用 MIT 许可证。

VDMS 支持：
* K 最近邻搜索
* 欧几里得距离 (L2) 和内积 (IP)
* 用于索引和计算距离的库：TileDBDense, TileDBSparse, FaissFlat (默认), FaissIVFFlat, Flinng
* 文本、图像和视频的嵌入
* 向量和元数据搜索

VDMS 具有服务器和客户端组件。要设置服务器，请参见 [安装说明](https://github.com/IntelLabs/vdms/blob/master/INSTALL.md) 或使用 [docker 镜像](https://hub.docker.com/r/intellabs/vdms)。

本笔记本展示了如何使用 docker 镜像将 VDMS 用作向量存储。

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 以使用此集成。

首先，安装 VDMS 客户端和句子变换器的 Python 包：

```python
# Pip install necessary package
%pip install --upgrade --quiet pip vdms sentence-transformers langchain-huggingface > /dev/null
```
```output
Note: you may need to restart the kernel to use updated packages.
```

## 启动 VDMS 服务器
在这里，我们使用端口 55555 启动 VDMS 服务器。

```python
!docker run --rm -d -p 55555:55555 --name vdms_vs_test_nb intellabs/vdms:latest
```
```output
b26917ffac236673ef1d035ab9c91fe999e29c9eb24aa6c7103d7baa6bf2f72d
```

## 基本示例（使用 Docker 容器）

在这个基本示例中，我们演示了如何将文档添加到 VDMS 并将其用作向量数据库。

您可以单独在 Docker 容器中运行 VDMS 服务器，以便与通过 VDMS Python 客户端连接到服务器的 LangChain 一起使用。

VDMS 具有处理多个文档集合的能力，但 LangChain 接口期望只有一个，因此我们需要指定集合的名称。LangChain 使用的默认集合名称是 "langchain"。

```python
import time
import warnings

warnings.filterwarnings("ignore")

from langchain_community.document_loaders.text import TextLoader
from langchain_community.vectorstores import VDMS
from langchain_community.vectorstores.vdms import VDMS_Client
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_text_splitters.character import CharacterTextSplitter

time.sleep(2)
DELIMITER = "-" * 50

# Connect to VDMS Vector Store
vdms_client = VDMS_Client(host="localhost", port=55555)
```

以下是一些用于打印结果的辅助函数。

```python
def print_document_details(doc):
    print(f"Content:\n\t{doc.page_content}\n")
    print("Metadata:")
    for key, value in doc.metadata.items():
        if value != "Missing property":
            print(f"\t{key}:\t{value}")


def print_results(similarity_results, score=True):
    print(f"{DELIMITER}\n")
    if score:
        for doc, score in similarity_results:
            print(f"Score:\t{score}\n")
            print_document_details(doc)
            print(f"{DELIMITER}\n")
    else:
        for doc in similarity_results:
            print_document_details(doc)
            print(f"{DELIMITER}\n")


def print_response(list_of_entities):
    for ent in list_of_entities:
        for key, value in ent.items():
            if value != "Missing property":
                print(f"\n{key}:\n\t{value}")
        print(f"{DELIMITER}\n")
```

### 加载文档并获取嵌入函数
在这里，我们加载最新的国情咨文并将文档拆分为多个块。

LangChain 向量存储使用字符串/关键字 `id` 来记录文档。默认情况下，`id` 是 uuid，但在这里我们将其定义为转换为字符串的整数。文档还提供了额外的元数据，并且在此示例中使用 HuggingFaceEmbeddings 作为嵌入函数。

```python
# load the document and split it into chunks
document_path = "../../how_to/state_of_the_union.txt"
raw_documents = TextLoader(document_path).load()

# split it into chunks
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(raw_documents)
ids = []
for doc_idx, doc in enumerate(docs):
    ids.append(str(doc_idx + 1))
    docs[doc_idx].metadata["id"] = str(doc_idx + 1)
    docs[doc_idx].metadata["page_number"] = int(doc_idx + 1)
    docs[doc_idx].metadata["president_included"] = (
        "president" in doc.page_content.lower()
    )
print(f"# Documents: {len(docs)}")


# create the open-source embedding function
embedding = HuggingFaceEmbeddings()
print(
    f"# Embedding Dimensions: {len(embedding.embed_query('This is a test document.'))}"
)
```
```output
# Documents: 42
# Embedding Dimensions: 768
```

### 使用 Faiss Flat 和欧几里得距离（默认）的相似性搜索

在本节中，我们使用 FAISS IndexFlat 索引（默认）和欧几里得距离（默认）作为相似性搜索的距离度量，将文档添加到 VDMS。我们搜索与查询 `What did the president say about Ketanji Brown Jackson` 相关的三个文档（`k=3`）。

```python
# add data
collection_name = "my_collection_faiss_L2"
db_FaissFlat = VDMS.from_documents(
    docs,
    client=vdms_client,
    ids=ids,
    collection_name=collection_name,
    embedding=embedding,
)

# Query (No metadata filtering)
k = 3
query = "What did the president say about Ketanji Brown Jackson"
returned_docs = db_FaissFlat.similarity_search(query, k=k, filter=None)
print_results(returned_docs, score=False)
```


```python
# Query (with filtering)
k = 3
constraints = {"page_number": [">", 30], "president_included": ["==", True]}
query = "What did the president say about Ketanji Brown Jackson"
returned_docs = db_FaissFlat.similarity_search(query, k=k, filter=constraints)
print_results(returned_docs, score=False)
```

### 使用 Faiss IVFFlat 和内积 (IP) 距离进行相似性搜索

在本节中，我们使用 Faiss IndexIVFFlat 索引将文档添加到 VDMS，并使用 IP 作为相似性搜索的距离度量。我们搜索与查询 `What did the president say about Ketanji Brown Jackson` 相关的三个文档 (`k=3`)，并返回文档及其分数。



```python
db_FaissIVFFlat = VDMS.from_documents(
    docs,
    client=vdms_client,
    ids=ids,
    collection_name="my_collection_FaissIVFFlat_IP",
    embedding=embedding,
    engine="FaissIVFFlat",
    distance_strategy="IP",
)
# Query
k = 3
query = "What did the president say about Ketanji Brown Jackson"
docs_with_score = db_FaissIVFFlat.similarity_search_with_score(query, k=k, filter=None)
print_results(docs_with_score)
```

### 使用 FLINNG 和 IP 距离的相似性搜索

在本节中，我们使用过滤器来识别近邻组（FLINNG）索引并使用 IP 作为相似性搜索的距离度量将文档添加到 VDMS。我们搜索与查询 `What did the president say about Ketanji Brown Jackson` 相关的三份文档（`k=3`），并返回分数和文档。

```python
db_Flinng = VDMS.from_documents(
    docs,
    client=vdms_client,
    ids=ids,
    collection_name="my_collection_Flinng_IP",
    embedding=embedding,
    engine="Flinng",
    distance_strategy="IP",
)
# Query
k = 3
query = "What did the president say about Ketanji Brown Jackson"
docs_with_score = db_Flinng.similarity_search_with_score(query, k=k, filter=None)
print_results(docs_with_score)
```

### 使用TileDBDense和欧几里得距离进行相似性搜索

在本节中，我们使用TileDB Dense索引和L2作为相似性搜索的距离度量，将文档添加到VDMS。我们搜索与查询`What did the president say about Ketanji Brown Jackson`相关的三个文档（`k=3`），并返回得分和文档。

```python
db_tiledbD = VDMS.from_documents(
    docs,
    client=vdms_client,
    ids=ids,
    collection_name="my_collection_tiledbD_L2",
    embedding=embedding,
    engine="TileDBDense",
    distance_strategy="L2",
)

k = 3
query = "What did the president say about Ketanji Brown Jackson"
docs_with_score = db_tiledbD.similarity_search_with_score(query, k=k, filter=None)
print_results(docs_with_score)
```

### 更新和删除

在构建真实应用程序的过程中，您希望超越添加数据，还要更新和删除数据。

以下是一个基本示例，展示如何做到这一点。首先，我们将通过添加日期来更新与查询最相关的文档的元数据。

```python
from datetime import datetime

doc = db_FaissFlat.similarity_search(query)[0]
print(f"Original metadata: \n\t{doc.metadata}")

# Update the metadata for a document by adding last datetime document read
datetime_str = datetime(2024, 5, 1, 14, 30, 0).isoformat()
doc.metadata["last_date_read"] = {"_date": datetime_str}
print(f"new metadata: \n\t{doc.metadata}")
print(f"{DELIMITER}\n")

# Update document in VDMS
id_to_update = doc.metadata["id"]
db_FaissFlat.update_document(collection_name, id_to_update, doc)
response, response_array = db_FaissFlat.get(
    collection_name,
    constraints={
        "id": ["==", id_to_update],
        "last_date_read": [">=", {"_date": "2024-05-01T00:00:00"}],
    },
)

# Display Results
print(f"UPDATED ENTRY (id={id_to_update}):")
print_response([response[0]["FindDescriptor"]["entities"][0]])
```

接下来，我们将通过 ID（id=42）删除最后一个文档。

```python
print("Documents before deletion: ", db_FaissFlat.count(collection_name))

id_to_remove = ids[-1]
db_FaissFlat.delete(collection_name=collection_name, ids=[id_to_remove])
print(
    f"Documents after deletion (id={id_to_remove}): {db_FaissFlat.count(collection_name)}"
)
```
```output
Documents before deletion:  42
Documents after deletion (id=42): 41
```

## 其他信息
VDMS 支持多种类型的视觉数据和操作。部分功能已集成在 LangChain 接口中，但随着 VDMS 的持续开发，将会增加更多的工作流程改进。

集成到 LangChain 中的其他功能如下。

### 基于向量的相似性搜索
除了通过字符串查询进行搜索外，您还可以通过嵌入/向量进行搜索。

```python
embedding_vector = embedding.embed_query(query)
returned_docs = db_FaissFlat.similarity_search_by_vector(embedding_vector)

# Print Results
print_document_details(returned_docs[0])
```
```output
Content:
	Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.

Metadata:
	id:	32
	last_date_read:	2024-05-01T14:30:00+00:00
	page_number:	32
	president_included:	True
	source:	../../how_to/state_of_the_union.txt
```

### 基于元数据的过滤

在处理集合之前，缩小范围可能会有所帮助。

例如，可以使用 get 方法根据元数据过滤集合。使用字典来过滤元数据。在这里，我们检索 `id = 2` 的文档并将其从向量存储中移除。

```python
response, response_array = db_FaissFlat.get(
    collection_name,
    limit=1,
    include=["metadata", "embeddings"],
    constraints={"id": ["==", "2"]},
)

# Delete id=2
db_FaissFlat.delete(collection_name=collection_name, ids=["2"])

print("Deleted entry:")
print_response([response[0]["FindDescriptor"]["entities"][0]])
```
```output
Deleted entry:

blob:
	True

content:
	Groups of citizens blocking tanks with their bodies. Everyone from students to retirees teachers turned soldiers defending their homeland. 

In this struggle as President Zelenskyy said in his speech to the European Parliament “Light will win over darkness.” The Ukrainian Ambassador to the United States is here tonight. 

Let each of us here tonight in this Chamber send an unmistakable signal to Ukraine and to the world. 

Please rise if you are able and show that, Yes, we the United States of America stand with the Ukrainian people. 

Throughout our history we’ve learned this lesson when dictators do not pay a price for their aggression they cause more chaos.   

They keep moving.   

And the costs and the threats to America and the world keep rising.   

That’s why the NATO Alliance was created to secure peace and stability in Europe after World War 2. 

The United States is a member along with 29 other nations. 

It matters. American diplomacy matters. American resolve matters.

id:
	2

page_number:
	2

president_included:
	True

source:
	../../how_to/state_of_the_union.txt
--------------------------------------------------
```

### 检索器选项

本节介绍如何将 VDMS 用作检索器的不同选项。


#### 相似性搜索

在这里，我们在检索器对象中使用相似性搜索。



```python
retriever = db_FaissFlat.as_retriever()
relevant_docs = retriever.invoke(query)[0]

print_document_details(relevant_docs)
```
```output
Content:
	Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.

Metadata:
	id:	32
	last_date_read:	2024-05-01T14:30:00+00:00
	page_number:	32
	president_included:	True
	source:	../../how_to/state_of_the_union.txt
```
#### 最大边际相关性搜索 (MMR)

除了在检索器对象中使用相似性搜索外，您还可以使用 `mmr`。


```python
retriever = db_FaissFlat.as_retriever(search_type="mmr")
relevant_docs = retriever.invoke(query)[0]

print_document_details(relevant_docs)
```
```output
Content:
	Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.

Metadata:
	id:	32
	last_date_read:	2024-05-01T14:30:00+00:00
	page_number:	32
	president_included:	True
	source:	../../how_to/state_of_the_union.txt
```
我们还可以直接使用 MMR。


```python
mmr_resp = db_FaissFlat.max_marginal_relevance_search_with_score(query, k=2, fetch_k=10)
print_results(mmr_resp)
```

### 删除集合
之前，我们是根据 `id` 删除文档。在这里，由于未提供 ID，所有文档都被删除。


```python
print("Documents before deletion: ", db_FaissFlat.count(collection_name))

db_FaissFlat.delete(collection_name=collection_name)

print("Documents after deletion: ", db_FaissFlat.count(collection_name))
```
```output
Documents before deletion:  40
Documents after deletion:  0
```

## 停止 VDMS 服务器


```python
!docker kill vdms_vs_test_nb
```
```output
huggingface/tokenizers: 当前进程刚刚被分叉，已经使用了并行性。禁用并行性以避免死锁...
要禁用此警告，您可以：
	- 如果可能，避免在分叉之前使用 `tokenizers`
	- 明确设置环境变量 TOKENIZERS_PARALLELISM=(true | false)
``````output
vdms_vs_test_nb
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)