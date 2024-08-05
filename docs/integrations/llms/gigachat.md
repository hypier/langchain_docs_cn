---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/gigachat.ipynb
---

# GigaChat
本笔记本展示了如何将 LangChain 与 [GigaChat](https://developers.sber.ru/portal/products/gigachat) 结合使用。
要使用该功能，您需要安装 ```gigachat``` python 包。

```python
%pip install --upgrade --quiet  gigachat
```

要获取 GigaChat 凭证，您需要 [创建账户](https://developers.sber.ru/studio/login) 并 [获取 API 访问权限](https://developers.sber.ru/docs/ru/gigachat/individuals-quickstart)

## 示例


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

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)