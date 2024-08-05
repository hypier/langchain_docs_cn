---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/aleph_alpha.ipynb
---

# Aleph Alpha

有两种可能的方式来使用 Aleph Alpha 的语义嵌入。如果您有结构不相似的文本（例如，一个文档和一个查询），您会想使用非对称嵌入。相反，对于结构相似的文本，建议使用对称嵌入。

## 非对称


```python
from langchain_community.embeddings import AlephAlphaAsymmetricSemanticEmbedding
```


```python
document = "This is a content of the document"
query = "What is the content of the document?"
```


```python
embeddings = AlephAlphaAsymmetricSemanticEmbedding(normalize=True, compress_to_size=128)
```


```python
doc_result = embeddings.embed_documents([document])
```


```python
query_result = embeddings.embed_query(query)
```

## 对称

```python
from langchain_community.embeddings import AlephAlphaSymmetricSemanticEmbedding
```

```python
text = "This is a test text"
```

```python
embeddings = AlephAlphaSymmetricSemanticEmbedding(normalize=True, compress_to_size=128)
```

```python
doc_result = embeddings.embed_documents([text])
```

```python
query_result = embeddings.embed_query(text)
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)