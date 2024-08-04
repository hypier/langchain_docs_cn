---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/aleph_alpha.ipynb
---
# Aleph Alpha

[The Luminous series](https://docs.aleph-alpha.com/docs/introduction/luminous/) is a family of large language models.

This example goes over how to use LangChain to interact with Aleph Alpha models


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



## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)