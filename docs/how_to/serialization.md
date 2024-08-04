---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/serialization.ipynb
---
# How to save and load LangChain objects

LangChain classes implement standard methods for serialization. Serializing LangChain objects using these methods confer some advantages:

- Secrets, such as API keys, are separated from other parameters and can be loaded back to the object on de-serialization;
- De-serialization is kept compatible across package versions, so objects that were serialized with one version of LangChain can be properly de-serialized with another.

To save and load LangChain objects using this system, use the `dumpd`, `dumps`, `load`, and `loads` functions in the [load module](https://api.python.langchain.com/en/latest/core_api_reference.html#module-langchain_core.load) of `langchain-core`. These functions support JSON and JSON-serializable objects.

All LangChain objects that inherit from [Serializable](https://api.python.langchain.com/en/latest/load/langchain_core.load.serializable.Serializable.html) are JSON-serializable. Examples include [messages](https://api.python.langchain.com/en/latest/core_api_reference.html#module-langchain_core.messages), [document objects](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html) (e.g., as returned from [retrievers](/docs/concepts/#retrievers)), and most [Runnables](/docs/concepts/#langchain-expression-language-lcel), such as chat models, retrievers, and [chains](/docs/how_to/sequence) implemented with the LangChain Expression Language.

Below we walk through an example with a simple [LLM chain](/docs/tutorials/llm_chain).

:::caution

De-serialization using `load` and `loads` can instantiate any serializable LangChain object. Only use this feature with trusted inputs!

De-serialization is a beta feature and is subject to change.
:::


```python
from langchain_core.load import dumpd, dumps, load, loads
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "Translate the following into {language}:"),
        ("user", "{text}"),
    ],
)

llm = ChatOpenAI(model="gpt-3.5-turbo-0125", api_key="llm-api-key")

chain = prompt | llm
```

## Saving objects

### To json


```python
string_representation = dumps(chain, pretty=True)
print(string_representation[:500])
```
```output
{
  "lc": 1,
  "type": "constructor",
  "id": [
    "langchain",
    "schema",
    "runnable",
    "RunnableSequence"
  ],
  "kwargs": {
    "first": {
      "lc": 1,
      "type": "constructor",
      "id": [
        "langchain",
        "prompts",
        "chat",
        "ChatPromptTemplate"
      ],
      "kwargs": {
        "input_variables": [
          "language",
          "text"
        ],
        "messages": [
          {
            "lc": 1,
            "type": "constructor",
```
### To a json-serializable Python dict


```python
dict_representation = dumpd(chain)

print(type(dict_representation))
```
```output
<class 'dict'>
```
### To disk


```python
import json

with open("/tmp/chain.json", "w") as fp:
    json.dump(string_representation, fp)
```

Note that the API key is withheld from the serialized representations. Parameters that are considered secret are specified by the `.lc_secrets` attribute of the LangChain object:


```python
chain.last.lc_secrets
```



```output
{'openai_api_key': 'OPENAI_API_KEY'}
```


## Loading objects

Specifying `secrets_map` in `load` and `loads` will load the corresponding secrets onto the de-serialized LangChain object.

### From string


```python
chain = loads(string_representation, secrets_map={"OPENAI_API_KEY": "llm-api-key"})
```

### From dict


```python
chain = load(dict_representation, secrets_map={"OPENAI_API_KEY": "llm-api-key"})
```

### From disk


```python
with open("/tmp/chain.json", "r") as fp:
    chain = loads(json.load(fp), secrets_map={"OPENAI_API_KEY": "llm-api-key"})
```

Note that we recover the API key specified at the start of the guide:


```python
chain.last.openai_api_key.get_secret_value()
```



```output
'llm-api-key'
```

