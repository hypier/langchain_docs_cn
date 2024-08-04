---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/fireworks.ipynb
---
# FireworksEmbeddings

This notebook explains how to use Fireworks Embeddings, which is included in the langchain_fireworks package, to embed texts in langchain. We use the default nomic-ai v1.5 model in this example.


```python
%pip install -qU langchain-fireworks
```

## Setup


```python
from langchain_fireworks import FireworksEmbeddings
```


```python
import getpass
import os

if "FIREWORKS_API_KEY" not in os.environ:
    os.environ["FIREWORKS_API_KEY"] = getpass.getpass("Fireworks API Key:")
```

# Using the Embedding Model
With `FireworksEmbeddings`, you can directly use the default model 'nomic-ai/nomic-embed-text-v1.5', or set a different one if available.


```python
embedding = FireworksEmbeddings(model="nomic-ai/nomic-embed-text-v1.5")
```


```python
res_query = embedding.embed_query("The test information")
res_document = embedding.embed_documents(["test1", "another test"])
print(res_query[:5])
print(res_document[1][:5])
```
```output
[0.01367950439453125, 0.0103607177734375, -0.157958984375, -0.003070831298828125, 0.05926513671875]
[0.0369873046875, 0.00545501708984375, -0.179931640625, -0.018707275390625, 0.0552978515625]
```

## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
