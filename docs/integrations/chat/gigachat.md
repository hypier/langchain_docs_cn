---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/gigachat.ipynb
---
# GigaChat
This notebook shows how to use LangChain with [GigaChat](https://developers.sber.ru/portal/products/gigachat).
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
from langchain_community.chat_models import GigaChat

chat = GigaChat(verify_ssl_certs=False, scope="GIGACHAT_API_PERS")
```


```python
from langchain_core.messages import HumanMessage, SystemMessage

messages = [
    SystemMessage(
        content="You are a helpful AI that shares everything you know. Talk in English."
    ),
    HumanMessage(content="What is capital of Russia?"),
]

print(chat.invoke(messages).content)
```
```output
The capital of Russia is Moscow.
```

## Related

- Chat model [conceptual guide](/docs/concepts/#chat-models)
- Chat model [how-to guides](/docs/how_to/#chat-models)