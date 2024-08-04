---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/anthropic.ipynb
sidebar_label: Anthropic
sidebar_class_name: hidden
---
# AnthropicLLM

:::caution
You are currently on a page documenting the use of Anthropic legacy Claude 2 models as [text completion models](/docs/concepts/#llms). The latest and most popular Anthropic models are [chat completion models](/docs/concepts/#chat-models).

You are probably looking for [this page instead](/docs/integrations/chat/anthropic/).
:::

This example goes over how to use LangChain to interact with `Anthropic` models.

## Installation


```python
%pip install -qU langchain-anthropic
```

## Environment Setup

We'll need to get an [Anthropic](https://console.anthropic.com/settings/keys) API key and set the `ANTHROPIC_API_KEY` environment variable:


```python
import os
from getpass import getpass

os.environ["ANTHROPIC_API_KEY"] = getpass()
```

## Usage


```python
from langchain_anthropic import AnthropicLLM
from langchain_core.prompts import PromptTemplate

template = """Question: {question}

Answer: Let's think step by step."""

prompt = PromptTemplate.from_template(template)

model = AnthropicLLM(model="claude-2.1")

chain = prompt | model

chain.invoke({"question": "What is LangChain?"})
```



```output
'\nLangChain is a decentralized blockchain network that leverages AI and machine learning to provide language translation services.'
```



## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)