---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/aleph_alpha.ipynb
---

# Aleph Alpha

[The Luminous series](https://docs.aleph-alpha.com/docs/introduction/luminous/) 是一系列大型语言模型。

这个示例介绍了如何使用 LangChain 与 Aleph Alpha 模型进行交互


```python
# Installing the langchain package needed to use the integration
%pip install -qU langchain-community
```


```python
# Install the package
%pip install --upgrade --quiet  aleph-alpha-client
```


```python
# create a new token: https://docs.aleph-alpha.com/docs/account/#create-a-new-token

from getpass import getpass

ALEPH_ALPHA_API_KEY = getpass()
```
```output
········
```

```python
from langchain_community.llms import AlephAlpha
from langchain_core.prompts import PromptTemplate
```


```python
template = """Q: {question}

A:"""

prompt = PromptTemplate.from_template(template)
```


```python
llm = AlephAlpha(
    model="luminous-extended",
    maximum_tokens=20,
    stop_sequences=["Q:"],
    aleph_alpha_api_key=ALEPH_ALPHA_API_KEY,
)
```


```python
llm_chain = prompt | llm
```


```python
question = "What is AI?"

llm_chain.invoke({"question": question})
```



```output
' Artificial Intelligence is the simulation of human intelligence processes by machines.\n\n'
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)