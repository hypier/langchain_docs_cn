---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/nanopq.ipynb
---

# NanoPQ (产品量化)

>[产品量化算法 (k-NN)](https://towardsdatascience.com/similarity-search-product-quantization-b2a1a6397701) 简要来说，是一种量化算法，帮助压缩数据库向量，在涉及大数据集时有助于语义搜索。简而言之，嵌入被分割成 M 个子空间，进一步进行聚类。在聚类向量后，质心向量被映射到子空间中每个聚类的向量上。

本笔记本讲解如何使用一个检索器，该检索器在底层使用了由 [nanopq](https://github.com/matsui528/nanopq) 包实现的产品量化。

```python
%pip install -qU langchain-community langchain-openai nanopq
```

```python
from langchain_community.embeddings.spacy_embeddings import SpacyEmbeddings
from langchain_community.retrievers import NanoPQRetriever
```

## 使用文本创建新的检索器


```python
retriever = NanoPQRetriever.from_texts(
    ["Great world", "great words", "world", "planets of the world"],
    SpacyEmbeddings(model_name="en_core_web_sm"),
    clusters=2,
    subspace=2,
)
```

## 使用检索器

我们现在可以使用检索器了！


```python
retriever.invoke("earth")
```
```output
M: 2, Ks: 2, metric : <class 'numpy.uint8'>, code_dtype: l2
iter: 20, seed: 123
Training the subspace: 0 / 2
Training the subspace: 1 / 2
Encoding the subspace: 0 / 2
Encoding the subspace: 1 / 2
```


```output
[Document(page_content='world'),
 Document(page_content='Great world'),
 Document(page_content='great words'),
 Document(page_content='planets of the world')]
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)