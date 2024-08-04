---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/google_vertex_ai_palm.ipynb
---
# Google Vertex AI PaLM 

>[Vertex AI PaLM API](https://cloud.google.com/vertex-ai/docs/generative-ai/learn/overview) is a service on Google Cloud exposing the embedding models. 

Note: This integration is separate from the Google PaLM integration.

By default, Google Cloud [does not use](https://cloud.google.com/vertex-ai/docs/generative-ai/data-governance#foundation_model_development) Customer Data to train its foundation models as part of Google Cloud`s AI/ML Privacy Commitment. More details about how Google processes data can also be found in [Google's Customer Data Processing Addendum (CDPA)](https://cloud.google.com/terms/data-processing-addendum).

To use Vertex AI PaLM you must have the `langchain-google-vertexai` Python package installed and either:
- Have credentials configured for your environment (gcloud, workload identity, etc...)
- Store the path to a service account JSON file as the GOOGLE_APPLICATION_CREDENTIALS environment variable

This codebase uses the `google.auth` library which first looks for the application credentials variable mentioned above, and then looks for system-level auth.

For more information, see: 
- https://cloud.google.com/docs/authentication/application-default-credentials#GAC
- https://googleapis.dev/python/google-auth/latest/reference/google.auth.html#module-google.auth




```python
%pip install --upgrade --quiet langchain langchain-google-vertexai
```


```python
from langchain_google_vertexai import VertexAIEmbeddings
```


```python
embeddings = VertexAIEmbeddings()
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
