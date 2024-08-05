---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/together.ipynb
sidebar_label: Together AI
---

# TogetherEmbeddings

本笔记本介绍如何开始使用托管在 Together AI API 中的开源嵌入模型。

## 安装


```python
# install package
%pip install --upgrade --quiet  langchain-together
```

## 环境设置

请确保设置以下环境变量：

- `TOGETHER_API_KEY`

## 用法

首先，从 [此列表](https://docs.together.ai/docs/embedding-models) 中选择一个支持的模型。在以下示例中，我们将使用 `togethercomputer/m2-bert-80M-8k-retrieval`。

```python
from langchain_together.embeddings import TogetherEmbeddings

embeddings = TogetherEmbeddings(model="togethercomputer/m2-bert-80M-8k-retrieval")
```

```python
embeddings.embed_query("My query to look up")
```

```python
embeddings.embed_documents(
    ["This is a content of the document", "This is another document"]
)
```

```python
# async embed query
await embeddings.aembed_query("My query to look up")
```

```python
# async embed documents
await embeddings.aembed_documents(
    ["This is a content of the document", "This is another document"]
)
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)