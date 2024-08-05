---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/ensemble_retriever.ipynb
---

# 如何结合多个检索器的结果

[EnsembleRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain.retrievers.ensemble.EnsembleRetriever.html) 支持多个检索器结果的集成。它通过一个 [BaseRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain_core.retrievers.BaseRetriever.html) 对象的列表进行初始化。EnsembleRetrievers 根据 [Reciprocal Rank Fusion](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf) 算法对组成检索器的结果进行重新排序。

通过利用不同算法的优势，`EnsembleRetriever` 可以实现比任何单一算法更好的性能。

最常见的模式是将稀疏检索器（如 BM25）与密集检索器（如嵌入相似度）结合，因为它们的优势是互补的。这也被称为“混合检索”。稀疏检索器擅长根据关键词查找相关文档，而密集检索器则擅长根据语义相似性查找相关文档。

## 基本用法

下面我们展示如何将 [BM25Retriever](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.bm25.BM25Retriever.html) 与来自 [FAISS 向量存储](https://api.python.langchain.com/en/latest/vectorstores/langchain_community.vectorstores.faiss.FAISS.html) 的检索器进行集成。

```python
%pip install --upgrade --quiet  rank_bm25 > /dev/null
```

```python
from langchain.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings

doc_list_1 = [
    "I like apples",
    "I like oranges",
    "Apples and oranges are fruits",
]

# 初始化 bm25 检索器和 faiss 检索器
bm25_retriever = BM25Retriever.from_texts(
    doc_list_1, metadatas=[{"source": 1}] * len(doc_list_1)
)
bm25_retriever.k = 2

doc_list_2 = [
    "You like apples",
    "You like oranges",
]

embedding = OpenAIEmbeddings()
faiss_vectorstore = FAISS.from_texts(
    doc_list_2, embedding, metadatas=[{"source": 2}] * len(doc_list_2)
)
faiss_retriever = faiss_vectorstore.as_retriever(search_kwargs={"k": 2})

# 初始化集成检索器
ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, faiss_retriever], weights=[0.5, 0.5]
)
```

```python
docs = ensemble_retriever.invoke("apples")
docs
```

```output
[Document(page_content='I like apples', metadata={'source': 1}),
 Document(page_content='You like apples', metadata={'source': 2}),
 Document(page_content='Apples and oranges are fruits', metadata={'source': 1}),
 Document(page_content='You like oranges', metadata={'source': 2})]
```

## 运行时配置

我们还可以使用 [configurable fields](/docs/how_to/configure) 在运行时配置各个检索器。下面我们专门更新 FAISS 检索器的 "top-k" 参数：

```python
from langchain_core.runnables import ConfigurableField

faiss_retriever = faiss_vectorstore.as_retriever(
    search_kwargs={"k": 2}
).configurable_fields(
    search_kwargs=ConfigurableField(
        id="search_kwargs_faiss",
        name="Search Kwargs",
        description="The search kwargs to use",
    )
)

ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, faiss_retriever], weights=[0.5, 0.5]
)
```

```python
config = {"configurable": {"search_kwargs_faiss": {"k": 1}}}
docs = ensemble_retriever.invoke("apples", config=config)
docs
```

```output
[Document(page_content='I like apples', metadata={'source': 1}),
 Document(page_content='You like apples', metadata={'source': 2}),
 Document(page_content='Apples and oranges are fruits', metadata={'source': 1})]
```

请注意，这仅从 FAISS 检索器返回一个来源，因为我们在运行时传入了相关配置。