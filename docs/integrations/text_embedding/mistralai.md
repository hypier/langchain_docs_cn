---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/mistralai.ipynb
---
# MistralAI

This notebook explains how to use MistralAIEmbeddings, which is included in the langchain_mistralai package, to embed texts in langchain.


```python
# pip install -U langchain-mistralai
```

## import the library


```python
from langchain_mistralai import MistralAIEmbeddings
```


```python
embedding = MistralAIEmbeddings(api_key="your-api-key")
```

# Using the Embedding Model
With `MistralAIEmbeddings`, you can directly use the default model 'mistral-embed', or set a different one if available.


```python
embedding.model = "mistral-embed"  # or your preferred model if available
```


```python
res_query = embedding.embed_query("The test information")
res_document = embedding.embed_documents(["test1", "another test"])
```


## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
