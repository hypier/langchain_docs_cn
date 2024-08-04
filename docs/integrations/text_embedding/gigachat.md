---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/gigachat.ipynb
---
# GigaChat
This notebook shows how to use LangChain with [GigaChat embeddings](https://developers.sber.ru/portal/products/gigachat).
To use you need to install ```gigachat``` python package.


```python
%pip install --upgrade --quiet  gigachat
```

To get GigaChat credentials you need to [create account](https://developers.sber.ru/studio/login) and [get access to API](https://developers.sber.ru/docs/ru/gigachat/individuals-quickstart)

## Example


```python
import os
from getpass import getpass

os.environ["GIGACHAT_CREDENTIALS"] = getpass()
```


```python
from langchain_community.embeddings import GigaChatEmbeddings

embeddings = GigaChatEmbeddings(verify_ssl_certs=False, scope="GIGACHAT_API_PERS")
```


```python
query_result = embeddings.embed_query("The quick brown fox jumps over the lazy dog")
```


```python
query_result[:5]
```



```output
[0.8398333191871643,
 -0.14180311560630798,
 -0.6161925792694092,
 -0.17103666067123413,
 1.2884578704833984]
```



## Related

- Embedding model [conceptual guide](/docs/concepts/#embedding-models)
- Embedding model [how-to guides](/docs/how_to/#embedding-models)
