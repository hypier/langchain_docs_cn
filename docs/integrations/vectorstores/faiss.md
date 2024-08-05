---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/faiss.ipynb
---

# Faiss

>[Facebook AI Similarity Search (Faiss)](https://engineering.fb.com/2017/03/29/data-infrastructure/faiss-a-library-for-efficient-similarity-search/) 是一个用于高效相似性搜索和密集向量聚类的库。它包含可以在任意大小的向量集合中进行搜索的算法，甚至可以处理可能不适合 RAM 的向量集合。它还包含用于评估和参数调整的支持代码。

[Faiss 文档](https://faiss.ai/)。

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 以使用此集成。

本笔记本展示了如何使用与 `FAISS` 向量数据库相关的功能。它将展示特定于此集成的功能。完成后，探索 [相关用例页面](/docs/how_to#qa-with-rag) 可能会有帮助，以了解如何将此向量存储作为更大链的一部分使用。

## 设置

集成位于 `langchain-community` 包中。我们还需要安装 `faiss` 包本身。我们还将使用 OpenAI 进行嵌入，因此需要安装这些依赖。我们可以使用以下命令进行安装：

```bash
pip install -U langchain-community faiss-cpu langchain-openai tiktoken
```

请注意，如果您想使用支持 GPU 的版本，也可以安装 `faiss-gpu`。

由于我们正在使用 OpenAI，您需要一个 OpenAI API 密钥。

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()
```

设置 [LangSmith](https://smith.langchain.com/) 以获得最佳的可观察性也是有帮助的（但不是必须的）。

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 数据摄取

在这里，我们将文档摄取到向量存储中


```python
# Uncomment the following line if you need to initialize FAISS with no AVX2 optimization
# os.environ['FAISS_NO_AVX2'] = '1'

from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
db = FAISS.from_documents(docs, embeddings)
print(db.index.ntotal)
```



```output
42
```

## 查询

现在，我们可以查询向量存储。有几种方法可以做到这一点。最标准的方法是使用 `similarity_search`。

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
```

```python
print(docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

## 作为检索器

我们还可以将向量存储转换为 [Retriever](/docs/how_to#retrievers) 类。这使我们能够轻松地在其他 LangChain 方法中使用它，这些方法主要与检索器一起工作。

```python
retriever = db.as_retriever()
docs = retriever.invoke(query)
```

```python
print(docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

## 相似性搜索与评分
有一些特定于 FAISS 的方法。其中之一是 `similarity_search_with_score`，它允许您不仅返回文档，还返回查询与文档之间的距离评分。返回的距离评分是 L2 距离。因此，分数越低越好。

```python
docs_and_scores = db.similarity_search_with_score(query)
```

```python
docs_and_scores[0]
```

```output
(Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../how_to/state_of_the_union.txt'}),
 0.36913747)
```

也可以使用 `similarity_search_by_vector` 对与给定嵌入向量相似的文档进行搜索，该方法接受嵌入向量作为参数，而不是字符串。

```python
embedding_vector = embeddings.embed_query(query)
docs_and_scores = db.similarity_search_by_vector(embedding_vector)
```

## 保存和加载
您还可以保存和加载 FAISS 索引。这很有用，这样您就不必每次使用时都重新创建它。

```python
db.save_local("faiss_index")

new_db = FAISS.load_local("faiss_index", embeddings)

docs = new_db.similarity_search(query)
```

```python
docs[0]
```

```output
Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../../state_of_the_union.txt'})
```

# 序列化和反序列化为字节

您可以通过这些函数对 FAISS 索引进行序列化。如果您使用的嵌入模型大小为 90 mb（sentence-transformers/all-MiniLM-L6-v2 或其他任何模型），则结果的 pickle 文件大小将超过 90 mb。模型的大小也包含在整体大小中。为了解决这个问题，请使用以下函数。这些函数仅序列化 FAISS 索引，大小会小得多。如果您希望将索引存储在像 SQL 这样的数据库中，这将非常有用。

```python
from langchain_huggingface import HuggingFaceEmbeddings

pkl = db.serialize_to_bytes()  # serializes the faiss
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")

db = FAISS.deserialize_from_bytes(
    embeddings=embeddings, serialized=pkl
)  # Load the index
```

## 合并
您还可以合并两个 FAISS 向量存储


```python
db1 = FAISS.from_texts(["foo"], embeddings)
db2 = FAISS.from_texts(["bar"], embeddings)

db1.docstore._dict
```


```python
db2.docstore._dict
```



```output
{'807e0c63-13f6-4070-9774-5c6f0fbb9866': Document(page_content='bar', metadata={})}
```



```python
db1.merge_from(db2)
```


```python
db1.docstore._dict
```



```output
{'068c473b-d420-487a-806b-fb0ccea7f711': Document(page_content='foo', metadata={}),
 '807e0c63-13f6-4070-9774-5c6f0fbb9866': Document(page_content='bar', metadata={})}
```

## 相似性搜索与过滤
FAISS vectorstore 也支持过滤，由于 FAISS 本身不支持过滤，我们需要手动进行。这是通过首先获取超过 `k` 的更多结果，然后进行过滤。这个过滤器可以是一个可调用对象，它接受一个元数据字典并返回一个布尔值，或者是一个元数据字典，其中每个缺失的键被忽略，每个存在的键必须在一个值的列表中。您还可以在调用任何搜索方法时设置 `fetch_k` 参数，以设置在过滤之前要获取多少文档。以下是一个小示例：

```python
from langchain_core.documents import Document

list_of_documents = [
    Document(page_content="foo", metadata=dict(page=1)),
    Document(page_content="bar", metadata=dict(page=1)),
    Document(page_content="foo", metadata=dict(page=2)),
    Document(page_content="barbar", metadata=dict(page=2)),
    Document(page_content="foo", metadata=dict(page=3)),
    Document(page_content="bar burr", metadata=dict(page=3)),
    Document(page_content="foo", metadata=dict(page=4)),
    Document(page_content="bar bruh", metadata=dict(page=4)),
]
db = FAISS.from_documents(list_of_documents, embeddings)
results_with_scores = db.similarity_search_with_score("foo")
for doc, score in results_with_scores:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}, Score: {score}")
```
```output
Content: foo, Metadata: {'page': 1}, Score: 5.159960813797904e-15
Content: foo, Metadata: {'page': 2}, Score: 5.159960813797904e-15
Content: foo, Metadata: {'page': 3}, Score: 5.159960813797904e-15
Content: foo, Metadata: {'page': 4}, Score: 5.159960813797904e-15
```
现在我们进行相同的查询调用，但我们只过滤 `page = 1` 

```python
results_with_scores = db.similarity_search_with_score("foo", filter=dict(page=1))
# 或者使用可调用对象：
# results_with_scores = db.similarity_search_with_score("foo", filter=lambda d: d["page"] == 1)
for doc, score in results_with_scores:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}, Score: {score}")
```
```output
Content: foo, Metadata: {'page': 1}, Score: 5.159960813797904e-15
Content: bar, Metadata: {'page': 1}, Score: 0.3131446838378906
```
同样的操作也可以在 `max_marginal_relevance_search` 中完成。

```python
results = db.max_marginal_relevance_search("foo", filter=dict(page=1))
for doc in results:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}")
```
```output
Content: foo, Metadata: {'page': 1}
Content: bar, Metadata: {'page': 1}
```
以下是如何在调用 `similarity_search` 时设置 `fetch_k` 参数的示例。通常，您希望 `fetch_k` 参数 >> `k` 参数。这是因为 `fetch_k` 参数是过滤之前将要获取的文档数量。如果将 `fetch_k` 设置为较小的数字，您可能无法获得足够的文档进行过滤。

```python
results = db.similarity_search("foo", filter=dict(page=1), k=1, fetch_k=4)
for doc in results:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}")
```
```output
Content: foo, Metadata: {'page': 1}
```

## 删除

您也可以从 vectorstore 中删除记录。在下面的示例中，`db.index_to_docstore_id` 代表一个包含 FAISS 索引元素的字典。

```python
print("count before:", db.index.ntotal)
db.delete([db.index_to_docstore_id[0]])
print("count after:", db.index.ntotal)
```



```output
count before: 8
count after: 7
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)