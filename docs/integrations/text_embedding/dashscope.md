---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/dashscope.ipynb
---
# DashScope

Let's load the DashScope Embedding class.


```python
from langchain_community.embeddings import DashScopeEmbeddings
```


```python
embeddings = DashScopeEmbeddings(
    model="text-embedding-v1", dashscope_api_key="your-dashscope-api-key"
)
```


```python
text = "This is a test document."
```


```python
query_result = embeddings.embed_query(text)
print(query_result)
```


```python
doc_results = embeddings.embed_documents(["foo"])
print(doc_results)
```


## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
