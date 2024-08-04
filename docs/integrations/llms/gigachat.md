---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/gigachat.ipynb
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
from langchain_community.llms import GigaChat

llm = GigaChat(verify_ssl_certs=False, scope="GIGACHAT_API_PERS")
```


```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate

template = "What is capital of {country}?"

prompt = PromptTemplate.from_template(template)

llm_chain = LLMChain(prompt=prompt, llm=llm)

generated = llm_chain.invoke(input={"country": "Russia"})
print(generated["text"])
```
```output
The capital of Russia is Moscow.
```

## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)
