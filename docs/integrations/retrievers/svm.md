---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/svm.ipynb
---

# SVM

>[支持向量机 (SVM)](https://scikit-learn.org/stable/modules/svm.html#support-vector-machines) 是一组用于分类、回归和异常值检测的监督学习方法。

本笔记本介绍如何使用一个底层使用 `SVM` 的检索器，该检索器使用 `scikit-learn` 包。

主要基于 https://github.com/karpathy/randomfun/blob/master/knn_vs_svm.html


```python
%pip install --upgrade --quiet  scikit-learn
```


```python
%pip install --upgrade --quiet  lark
```

我们想使用 `OpenAIEmbeddings`，因此我们需要获取 OpenAI API 密钥。


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```
```output
OpenAI API Key: ········
```

```python
from langchain_community.retrievers import SVMRetriever
from langchain_openai import OpenAIEmbeddings
```

## 使用文本创建新的检索器


```python
retriever = SVMRetriever.from_texts(
    ["foo", "bar", "world", "hello", "foo bar"], OpenAIEmbeddings()
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

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)