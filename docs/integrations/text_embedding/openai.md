---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/openai.ipynb
keywords: [openaiembeddings]
---
# OpenAI

Let's load the OpenAI Embedding class.

## Setup

First we install langchain-openai and set the required env vars


```python
%pip install -qU langchain-openai
```


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()
```


```python
from langchain_openai import OpenAIEmbeddings
```


```python
embeddings = OpenAIEmbeddings(model="text-embedding-3-large")
```


```python
text = "This is a test document."
```

## Usage
### Embed query


```python
query_result = embeddings.embed_query(text)
```
```output
Warning: model not found. Using cl100k_base encoding.
```

```python
query_result[:5]
```



```output
[-0.014380056377383358,
 -0.027191711627651764,
 -0.020042716111860304,
 0.057301379620345545,
 -0.022267658631828974]
```


## Embed documents


```python
doc_result = embeddings.embed_documents([text])
```
```output
Warning: model not found. Using cl100k_base encoding.
```

```python
doc_result[0][:5]
```



```output
[-0.014380056377383358,
 -0.027191711627651764,
 -0.020042716111860304,
 0.057301379620345545,
 -0.022267658631828974]
```


## Specify dimensions

With the `text-embedding-3` class of models, you can specify the size of the embeddings you want returned. For example by default `text-embedding-3-large` returned embeddings of dimension 3072:


```python
len(doc_result[0])
```



```output
3072
```


But by passing in `dimensions=1024` we can reduce the size of our embeddings to 1024:


```python
embeddings_1024 = OpenAIEmbeddings(model="text-embedding-3-large", dimensions=1024)
```


```python
len(embeddings_1024.embed_documents([text])[0])
```
```output
Warning: model not found. Using cl100k_base encoding.
```


```output
1024
```



## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
