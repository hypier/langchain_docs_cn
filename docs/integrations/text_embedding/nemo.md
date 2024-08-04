---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/nemo.ipynb
---
# NVIDIA NeMo embeddings

Connect to NVIDIA's embedding service using the `NeMoEmbeddings` class.

The NeMo Retriever Embedding Microservice (NREM) brings the power of state-of-the-art text embedding to your applications, providing unmatched natural language processing and understanding capabilities. Whether you're developing semantic search, Retrieval Augmented Generation (RAG) pipelines—or any application that needs to use text embeddings—NREM has you covered. Built on the NVIDIA software platform incorporating CUDA, TensorRT, and Triton, NREM brings state of the art GPU accelerated Text Embedding model serving.

NREM uses NVIDIA's TensorRT built on top of the Triton Inference Server for optimized inference of text embedding models.

## Imports


```python
from langchain_community.embeddings import NeMoEmbeddings
```

## Setup


```python
batch_size = 16
model = "NV-Embed-QA-003"
api_endpoint_url = "http://localhost:8080/v1/embeddings"
```


```python
embedding_model = NeMoEmbeddings(
    batch_size=batch_size, model=model, api_endpoint_url=api_endpoint_url
)
```
```output
Checking if endpoint is live: http://localhost:8080/v1/embeddings
```

```python
embedding_model.embed_query("This is a test.")
```


## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
