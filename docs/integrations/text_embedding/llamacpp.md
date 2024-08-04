---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/llamacpp.ipynb
---
# Llama-cpp

This notebook goes over how to use Llama-cpp embeddings within LangChain


```python
%pip install --upgrade --quiet  llama-cpp-python
```


```python
from langchain_community.embeddings import LlamaCppEmbeddings
```


```python
llama = LlamaCppEmbeddings(model_path="/path/to/model/ggml-model-q4_0.bin")
```


```python
text = "This is a test document."
```


```python
query_result = llama.embed_query(text)
```


```python
doc_result = llama.embed_documents([text])
```


## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
