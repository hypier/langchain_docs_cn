---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/together.ipynb
sidebar_label: Together AI
---
# TogetherEmbeddings

This notebook covers how to get started with open source embedding models hosted in the Together AI API.

## Installation


```python
# install package
%pip install --upgrade --quiet  langchain-together
```

## Environment Setup

Make sure to set the following environment variables:

- `TOGETHER_API_KEY`

## Usage

First, select a supported model from [this list](https://docs.together.ai/docs/embedding-models). In the following example, we will use `togethercomputer/m2-bert-80M-8k-retrieval`.


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


## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
