---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/cohere.ipynb
---
# Cohere

:::caution
You are currently on a page documenting the use of Cohere models as [text completion models](/docs/concepts/#llms). Many popular Cohere models are [chat completion models](/docs/concepts/#chat-models).

You may be looking for [this page instead](/docs/integrations/chat/cohere/).
:::

>[Cohere](https://cohere.ai/about) is a Canadian startup that provides natural language processing models that help companies improve human-machine interactions.

Head to the [API reference](https://api.python.langchain.com/en/latest/llms/langchain_community.llms.cohere.Cohere.html) for detailed documentation of all attributes and methods.

## Setup

The integration lives in the `langchain-community` package. We also need to install the `cohere` package itself. We can install these with:

```bash
pip install -U langchain-community langchain-cohere
```

We'll also need to get a [Cohere API key](https://cohere.com/) and set the `COHERE_API_KEY` environment variable:


```python
import getpass
import os

os.environ["COHERE_API_KEY"] = getpass.getpass()
```
```output
 ········
```
It's also helpful (but not needed) to set up [LangSmith](https://smith.langchain.com/) for best-in-class observability


```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## Usage

Cohere supports all [LLM](/docs/how_to#llms) functionality:


```python
from langchain_cohere import Cohere
from langchain_core.messages import HumanMessage
```


```python
model = Cohere(max_tokens=256, temperature=0.75)
```


```python
message = "Knock knock"
model.invoke(message)
```



```output
" Who's there?"
```



```python
await model.ainvoke(message)
```



```output
" Who's there?"
```



```python
for chunk in model.stream(message):
    print(chunk, end="", flush=True)
```
```output
 Who's there?
```

```python
model.batch([message])
```



```output
[" Who's there?"]
```


You can also easily combine with a prompt template for easy structuring of user input. We can do this using [LCEL](/docs/concepts#langchain-expression-language-lcel)


```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template("Tell me a joke about {topic}")
chain = prompt | model
```


```python
chain.invoke({"topic": "bears"})
```



```output
' Why did the teddy bear cross the road?\nBecause he had bear crossings.\n\nWould you like to hear another joke? '
```



## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)
