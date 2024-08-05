---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/tf_idf.ipynb
---

# TF-IDF

>[TF-IDF](https://scikit-learn.org/stable/modules/feature_extraction.html#tfidf-term-weighting) 是词频与逆文档频率的乘积。

本笔记本介绍如何使用一个在底层使用 [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) 的检索器，该检索器使用 `scikit-learn` 包。

有关 TF-IDF 详细信息，请参见 [这篇博客文章](https://medium.com/data-science-bootcamp/tf-idf-basics-of-information-retrieval-48de122b2a4c)。

```python
%pip install --upgrade --quiet  scikit-learn
```

```python
from langchain_community.retrievers import TFIDFRetriever
```

## 使用文本创建新检索器


```python
retriever = TFIDFRetriever.from_texts(["foo", "bar", "world", "hello", "foo bar"])
```

## 创建一个带文档的新检索器

您现在可以使用您创建的文档来创建一个新的检索器。

```python
from langchain_core.documents import Document

retriever = TFIDFRetriever.from_documents(
    [
        Document(page_content="foo"),
        Document(page_content="bar"),
        Document(page_content="world"),
        Document(page_content="hello"),
        Document(page_content="foo bar"),
    ]
)
```

## 使用检索器

我们现在可以使用检索器了！


```python
result = retriever.invoke("foo")
```


```python
result
```



```output
[Document(page_content='foo', metadata={}),
 Document(page_content='foo bar', metadata={}),
 Document(page_content='hello', metadata={}),
 Document(page_content='world', metadata={})]
```

## 保存和加载

您可以轻松地保存和加载这个检索器，这使得它在本地开发中非常方便！


```python
retriever.save_local("testing.pkl")
```


```python
retriever_copy = TFIDFRetriever.load_local("testing.pkl")
```


```python
retriever_copy.invoke("foo")
```



```output
[Document(page_content='foo', metadata={}),
 Document(page_content='foo bar', metadata={}),
 Document(page_content='hello', metadata={}),
 Document(page_content='world', metadata={})]
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)