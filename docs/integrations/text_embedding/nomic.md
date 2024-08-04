---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/nomic.ipynb
sidebar_label: Nomic
---
# NomicEmbeddings

This notebook covers how to get started with Nomic embedding models.

## Installation


```python
# install package
!pip install -U langchain-nomic
```

## Environment Setup

Make sure to set the following environment variables:

- `NOMIC_API_KEY`

## Usage


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
# async embed query
await embeddings.aembed_query("My query to look up")
```


```python
# async embed documents
await embeddings.aembed_documents(
    ["This is a content of the document", "This is another document"]
)
```

### Custom Dimensionality

Nomic's `nomic-embed-text-v1.5` model was [trained with Matryoshka learning](https://blog.nomic.ai/posts/nomic-embed-matryoshka) to enable variable-length embeddings with a single model. This means that you can specify the dimensionality of the embeddings at inference time. The model supports dimensionality from 64 to 768.


```python
embeddings = NomicEmbeddings(model="nomic-embed-text-v1.5", dimensionality=256)

embeddings.embed_query("My query to look up")
```


## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
