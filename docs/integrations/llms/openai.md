---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/openai.ipynb
---
# OpenAI

:::caution
You are currently on a page documenting the use of OpenAI [text completion models](/docs/concepts/#llms). The latest and most popular OpenAI models are [chat completion models](/docs/concepts/#chat-models).

Unless you are specifically using `gpt-3.5-turbo-instruct`, you are probably looking for [this page instead](/docs/integrations/chat/openai/).
:::

[OpenAI](https://platform.openai.com/docs/introduction) offers a spectrum of models with different levels of power suitable for different tasks.

This example goes over how to use LangChain to interact with `OpenAI` [models](https://platform.openai.com/docs/models)


```python
# get a token: https://platform.openai.com/account/api-keys

from getpass import getpass

OPENAI_API_KEY = getpass()
```


```python
import os

os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

Should you need to specify your organization ID, you can use the following cell. However, it is not required if you are only part of a single organization or intend to use your default organization. You can check your default organization [here](https://platform.openai.com/account/api-keys).

To specify your organization, you can use this:
```python
OPENAI_ORGANIZATION = getpass()

os.environ["OPENAI_ORGANIZATION"] = OPENAI_ORGANIZATION
```


```python
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
```


```python
template = """Question: {question}

Answer: Let's think step by step."""

prompt = PromptTemplate.from_template(template)
```


```python
llm = OpenAI()
```

If you manually want to specify your OpenAI API key and/or organization ID, you can use the following:
```python
llm = OpenAI(openai_api_key="YOUR_API_KEY", openai_organization="YOUR_ORGANIZATION_ID")
```
Remove the openai_organization parameter should it not apply to you.


```python
llm_chain = prompt | llm
```


```python
question = "What NFL team won the Super Bowl in the year Justin Beiber was born?"

llm_chain.invoke(question)
```



```output
' Justin Bieber was born on March 1, 1994. The Super Bowl is typically played in late January or early February. So, we need to look at the Super Bowl from 1994. In 1994, the Super Bowl was Super Bowl XXVIII, played on January 30, 1994. The winning team of that Super Bowl was the Dallas Cowboys.'
```


If you are behind an explicit proxy, you can specify the http_client to pass through


```python
pip install httpx

import httpx

openai = OpenAI(model_name="gpt-3.5-turbo-instruct", http_client=httpx.Client(proxies="http://proxy.yourcompany.com:8080"))
```


## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)
