---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/astradb.ipynb
---

# Astra DB

本页面提供了使用 [Astra DB](https://docs.datastax.com/en/astra/home/astra.html) 作为向量存储的快速入门指南。

> DataStax [Astra DB](https://docs.datastax.com/en/astra/home/astra.html) 是一个无服务器的向量数据库，基于 Apache Cassandra® 构建，并通过易于使用的 JSON API 方便地提供。

_注意：除了访问数据库外，还需要一个 OpenAI API 密钥才能运行完整示例。_

## 设置和一般依赖

使用该集成需要相应的 Python 包：

```python
pip install -qU langchain-astradb
```

_确保您已安装运行此演示所需的所有包：_

```python
pip install -qU langchain langchain-community langchain-openai datasets pypdf
```

### 导入依赖


```python
import os
from getpass import getpass

from astrapy.info import CollectionVectorServiceOptions
from datasets import load_dataset
from langchain_community.document_loaders import PyPDFLoader
from langchain_core.documents import Document
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
```

## 导入向量存储


```python
from langchain_astradb import AstraDBVectorStore
```

## DB 连接参数

这些信息可以在您的 Astra DB 仪表板上找到：

- API 端点看起来像 `https://01234567-89ab-cdef-0123-456789abcdef-us-east1.apps.astra.datastax.com`
- 令牌看起来像 `AstraCS:6gBhNmsk135....`
- 您可以选择提供一个 _命名空间_，例如 `my_namespace`


```python
ASTRA_DB_API_ENDPOINT = input("ASTRA_DB_API_ENDPOINT = ")
ASTRA_DB_APPLICATION_TOKEN = getpass("ASTRA_DB_APPLICATION_TOKEN = ")

desired_namespace = input("(optional) Namespace = ")
if desired_namespace:
    ASTRA_DB_KEYSPACE = desired_namespace
else:
    ASTRA_DB_KEYSPACE = None
```

## 创建向量存储

有两种方法可以创建 Astra DB 向量存储，它们在计算嵌入的方式上有所不同。

*显式嵌入*。您可以单独实例化一个 `langchain_core.embeddings.Embeddings` 类，并将其传递给 `AstraDBVectorStore` 构造函数，就像与大多数其他 LangChain 向量存储一样。

*集成嵌入计算*。另外，您可以使用 Astra DB 的 [Vectorize](https://www.datastax.com/blog/simplifying-vector-embedding-generation-with-astra-vectorize) 功能，在创建存储时简单地指定一个支持的嵌入模型名称。嵌入计算完全在数据库内部处理。 （要使用此方法，您必须为您的数据库启用所需的嵌入集成功能，如 [文档](https://docs.datastax.com/en/astra-db-serverless/databases/embedding-generation.html) 中所述。）

**请选定一种方法并仅运行相应的单元格。**

### 方法 1：显式提供嵌入

此演示将使用 OpenAI 嵌入模型：


```python
os.environ["OPENAI_API_KEY"] = getpass("OPENAI_API_KEY = ")
```


```python
my_embeddings = OpenAIEmbeddings()
```

现在您可以创建向量存储：


```python
vstore = AstraDBVectorStore(
    embedding=my_embeddings,
    collection_name="astra_vector_demo",
    api_endpoint=ASTRA_DB_API_ENDPOINT,
    token=ASTRA_DB_APPLICATION_TOKEN,
    namespace=ASTRA_DB_KEYSPACE,
)
```

### 方法二：使用 Astra Vectorize（集成在 Astra DB 中的嵌入）

这里假设您已经

- 在您的 Astra DB 组织中启用了 OpenAI 集成，
- 为该集成添加了名为 `"MY_OPENAI_API_KEY"` 的 API 密钥，并且
- 将其作用域设置为您正在使用的数据库。

有关更多详细信息，请参阅 [文档](https://docs.datastax.com/en/astra-db-serverless/integrations/embedding-providers/openai.html)。

```python
openai_vectorize_options = CollectionVectorServiceOptions(
    provider="openai",
    model_name="text-embedding-3-small",
    authentication={
        "providerKey": "MY_OPENAI_API_KEY",
    },
)

vstore = AstraDBVectorStore(
    collection_name="astra_vectorize_demo",
    api_endpoint=ASTRA_DB_API_ENDPOINT,
    token=ASTRA_DB_APPLICATION_TOKEN,
    namespace=ASTRA_DB_KEYSPACE,
    collection_vector_service_options=openai_vectorize_options,
)
```

## 加载数据集

将源数据集中的每个条目转换为 `Document`，然后将它们写入向量存储：

```python
philo_dataset = load_dataset("datastax/philosopher-quotes")["train"]

docs = []
for entry in philo_dataset:
    metadata = {"author": entry["author"]}
    doc = Document(page_content=entry["quote"], metadata=metadata)
    docs.append(doc)

inserted_ids = vstore.add_documents(docs)
print(f"\nInserted {len(inserted_ids)} documents.")
```

在上面，`metadata` 字典是从源数据创建的，并且是 `Document` 的一部分。

_注意：请查看 [Astra DB API 文档](https://docs.datastax.com/en/astra-serverless/docs/develop/dev-with-json.html#_json_api_limits) 以获取有效的元数据字段名称：某些字符是保留的，不能使用。_

添加更多条目，这次使用 `add_texts`：

```python
texts = ["I think, therefore I am.", "To the things themselves!"]
metadatas = [{"author": "descartes"}, {"author": "husserl"}]
ids = ["desc_01", "huss_xy"]

inserted_ids_2 = vstore.add_texts(texts=texts, metadatas=metadatas, ids=ids)
print(f"\nInserted {len(inserted_ids_2)} documents.")
```

_注意：您可能希望通过增加这些批量操作的并发级别来加快 `add_texts` 和 `add_documents` 的执行速度 - 请查看类构造函数中的 `*_concurrency` 参数和 `add_texts` 文档字符串以获取更多详细信息。根据网络和客户端机器的规格，您最佳的参数选择可能会有所不同。_

## 运行搜索

本节演示了元数据过滤和获取相似度分数的过程：


```python
results = vstore.similarity_search("Our life is what we make of it", k=3)
for res in results:
    print(f"* {res.page_content} [{res.metadata}]")
```


```python
results_filtered = vstore.similarity_search(
    "Our life is what we make of it",
    k=3,
    filter={"author": "plato"},
)
for res in results_filtered:
    print(f"* {res.page_content} [{res.metadata}]")
```


```python
results = vstore.similarity_search_with_score("Our life is what we make of it", k=3)
for res, score in results:
    print(f"* [SIM={score:3f}] {res.page_content} [{res.metadata}]")
```

### MMR (最大边际相关性) 搜索

_注意：MMR 搜索方法尚不支持使用 Astra Vectorize 构建的向量存储。_

```python
results = vstore.max_marginal_relevance_search(
    "Our life is what we make of it",
    k=3,
    filter={"author": "aristotle"},
)
for res in results:
    print(f"* {res.page_content} [{res.metadata}]")
```

### Async

请注意，Astra DB 向量存储原生支持所有完全异步的方法（`asimilarity_search`、`afrom_texts`、`adelete` 等），即不涉及线程包装。

## 删除存储的文档


```python
delete_1 = vstore.delete(inserted_ids[:3])
print(f"all_succeed={delete_1}")  # True, all documents deleted
```


```python
delete_2 = vstore.delete(inserted_ids[2:5])
print(f"some_succeeds={delete_2}")  # True, though some IDs were gone already
```

## 最小化的 RAG 链

接下来的单元将实现一个简单的 RAG 管道：
- 下载一个示例 PDF 文件并将其加载到存储中；
- 使用 LCEL（LangChain 表达式语言）创建一个 RAG 链，以向量存储为核心；
- 运行问答链。


```python
!curl -L \
    "https://github.com/awesome-astra/datasets/blob/main/demo-resources/what-is-philosophy/what-is-philosophy.pdf?raw=true" \
    -o "what-is-philosophy.pdf"
```


```python
pdf_loader = PyPDFLoader("what-is-philosophy.pdf")
splitter = RecursiveCharacterTextSplitter(chunk_size=512, chunk_overlap=64)
docs_from_pdf = pdf_loader.load_and_split(text_splitter=splitter)

print(f"Documents from PDF: {len(docs_from_pdf)}.")
inserted_ids_from_pdf = vstore.add_documents(docs_from_pdf)
print(f"Inserted {len(inserted_ids_from_pdf)} documents.")
```


```python
retriever = vstore.as_retriever(search_kwargs={"k": 3})

philo_template = """
You are a philosopher that draws inspiration from great thinkers of the past
to craft well-thought answers to user questions. Use the provided context as the basis
for your answers and do not make up new reasoning paths - just mix-and-match what you are given.
Your answers must be concise and to the point, and refrain from answering about other topics than philosophy.

CONTEXT:
{context}

QUESTION: {question}

YOUR ANSWER:"""

philo_prompt = ChatPromptTemplate.from_template(philo_template)

llm = ChatOpenAI()

chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | philo_prompt
    | llm
    | StrOutputParser()
)
```


```python
chain.invoke("How does Russel elaborate on Peirce's idea of the security blanket?")
```

欲了解更多信息，请查看使用 Astra DB 的完整 RAG 模板 [这里](https://github.com/langchain-ai/langchain/tree/master/templates/rag-astradb)。

## 清理

如果您想从您的 Astra DB 实例中完全删除集合，请运行此命令。

_(您将失去存储在其中的数据。)_

```python
vstore.delete_collection()
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)