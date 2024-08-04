---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/azureopenai.ipynb
keywords: [AzureOpenAIEmbeddings]
---
# Azure OpenAI

Let's load the Azure OpenAI Embedding class with environment variables set to indicate to use Azure endpoints.


```python
%pip install --upgrade --quiet langchain-openai
```


```python
import os

os.environ["AZURE_OPENAI_API_KEY"] = "..."
os.environ["AZURE_OPENAI_ENDPOINT"] = "https://<your-endpoint>.openai.azure.com/"
```


```python
from langchain_openai import AzureOpenAIEmbeddings

embeddings = AzureOpenAIEmbeddings(
    azure_deployment="<your-embeddings-deployment-name>",
    openai_api_version="2023-05-15",
)
```


```python
text = "this is a test document"
```


```python
query_result = embeddings.embed_query(text)
```


```python
doc_result = embeddings.embed_documents([text])
```


```python
doc_result[0][:5]
```



```output
[-0.012222584727053133,
 0.0072103982392216145,
 -0.014818063280923775,
 -0.026444746872933557,
 -0.0034330499700826883]
```


## [Legacy] When using `openai<1`


```python
# set the environment variables needed for openai package to know to reach out to azure
import os

os.environ["OPENAI_API_TYPE"] = "azure"
os.environ["OPENAI_API_BASE"] = "https://<your-endpoint.openai.azure.com/"
os.environ["OPENAI_API_KEY"] = "your AzureOpenAI key"
os.environ["OPENAI_API_VERSION"] = "2023-05-15"
```


```python
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(deployment="your-embeddings-deployment-name")
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
