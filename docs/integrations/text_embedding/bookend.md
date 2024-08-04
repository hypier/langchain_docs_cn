---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/bookend.ipynb
---
# Bookend AI

Let's load the Bookend AI Embeddings class.


```python
from langchain_community.embeddings import BookendEmbeddings
```


```python
embeddings = BookendEmbeddings(
    domain="your_domain",
    api_token="your_api_token",
    model_id="your_embeddings_model_id",
)
```


```python
text = "This is a test document."
```


```python
query_result = embeddings.embed_query(text)
```


```python
doc_result = embeddings.embed_documents([text])
```


## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
