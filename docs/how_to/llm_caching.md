---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/llm_caching.ipynb
---

# 如何缓存LLM响应

LangChain为LLM提供了一个可选的缓存层。这有两个好处：

它可以通过减少您向LLM提供者发出的API调用次数来节省费用，特别是当您经常请求相同的完成时。
它可以通过减少您向LLM提供者发出的API调用次数来加快您的应用程序速度。



```python
%pip install -qU langchain_openai langchain_community

import os
from getpass import getpass

os.environ["OPENAI_API_KEY"] = getpass()
# 请手动输入OpenAI密钥
```


```python
from langchain.globals import set_llm_cache
from langchain_openai import OpenAI

# 为了让缓存变得更加明显，我们使用一个较慢和较旧的模型。
# 缓存也支持较新的聊天模型。
llm = OpenAI(model="gpt-3.5-turbo-instruct", n=2, best_of=2)
```


```python
%%time
from langchain.cache import InMemoryCache

set_llm_cache(InMemoryCache())

# 第一次调用时，它还不在缓存中，因此应该需要更长时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 546 ms, sys: 379 ms, total: 925 ms
Wall time: 1.11 s
```


```output
"\nWhy don't scientists trust atoms?\n\nBecause they make up everything!"
```



```python
%%time
# 第二次调用时，它已经在缓存中，因此速度更快
llm.invoke("Tell me a joke")
```
```output
CPU times: user 192 µs, sys: 77 µs, total: 269 µs
Wall time: 270 µs
```


```output
"\nWhy don't scientists trust atoms?\n\nBecause they make up everything!"
```

## SQLite 缓存


```python
!rm .langchain.db
```


```python
# 我们可以使用 SQLite 缓存做同样的事情
from langchain_community.cache import SQLiteCache

set_llm_cache(SQLiteCache(database_path=".langchain.db"))
```


```python
%%time
# 第一次，它还不在缓存中，所以应该花费更长时间
llm.invoke("Tell me a joke")
```
```output
CPU times: user 10.6 ms, sys: 4.21 ms, total: 14.8 ms
Wall time: 851 ms
```


```output
"\n\nWhy don't scientists trust atoms?\n\nBecause they make up everything!"
```



```python
%%time
# 第二次它在缓存中，所以速度更快
llm.invoke("Tell me a joke")
```
```output
CPU times: user 59.7 ms, sys: 63.6 ms, total: 123 ms
Wall time: 134 ms
```


```output
"\n\nWhy don't scientists trust atoms?\n\nBecause they make up everything!"
```