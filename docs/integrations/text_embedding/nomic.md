---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/nomic.ipynb
sidebar_label: Nomic
---

# NomicEmbeddings

本笔记本介绍如何开始使用 Nomic 嵌入模型。

## 安装


```python
# install package
!pip install -U langchain-nomic
```

## 环境设置

确保设置以下环境变量：

- `NOMIC_API_KEY`

## 使用方法


```python
from langchain_nomic.embeddings import NomicEmbeddings

embeddings = NomicEmbeddings(model="nomic-embed-text-v1.5")
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
# 异步嵌入查询
await embeddings.aembed_query("My query to look up")
```


```python
# 异步嵌入文档
await embeddings.aembed_documents(
    ["This is a content of the document", "This is another document"]
)
```

### 自定义维度

Nomic的 `nomic-embed-text-v1.5` 模型经过 [Matryoshka 学习](https://blog.nomic.ai/posts/nomic-embed-matryoshka) 训练，能够使用单一模型实现可变长度的嵌入。这意味着您可以在推理时指定嵌入的维度。该模型支持的维度范围为 64 到 768。

```python
embeddings = NomicEmbeddings(model="nomic-embed-text-v1.5", dimensionality=256)

embeddings.embed_query("My query to look up")
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)