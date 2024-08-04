---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/fireworks.ipynb
---
# Fireworks

:::caution
You are currently on a page documenting the use of Fireworks models as [text completion models](/docs/concepts/#llms). Many popular Fireworks models are [chat completion models](/docs/concepts/#chat-models).

You may be looking for [this page instead](/docs/integrations/chat/fireworks/).
:::

>[Fireworks](https://app.fireworks.ai/) accelerates product development on generative AI by creating an innovative AI experiment and production platform. 

This example goes over how to use LangChain to interact with `Fireworks` models.


```python
%pip install -qU langchain-fireworks
```


```python
from langchain_fireworks import Fireworks
```

# Setup

1. Make sure the `langchain-fireworks` package is installed in your environment.
2. Sign in to [Fireworks AI](http://fireworks.ai) for the an API Key to access our models, and make sure it is set as the `FIREWORKS_API_KEY` environment variable.
3. Set up your model using a model id. If the model is not set, the default model is fireworks-llama-v2-7b-chat. See the full, most up-to-date model list on [fireworks.ai](https://fireworks.ai).


```python
import getpass
import os

from langchain_fireworks import Fireworks

if "FIREWORKS_API_KEY" not in os.environ:
    os.environ["FIREWORKS_API_KEY"] = getpass.getpass("Fireworks API Key:")

# Initialize a Fireworks model
llm = Fireworks(
    model="accounts/fireworks/models/mixtral-8x7b-instruct",
    base_url="https://api.fireworks.ai/inference/v1/completions",
)
```

# Calling the Model Directly

You can call the model directly with string prompts to get completions.


```python
# Single prompt
output = llm.invoke("Who's the best quarterback in the NFL?")
print(output)
```
```output

Even if Tom Brady wins today, he'd still have the same
```

```python
# Calling multiple prompts
output = llm.generate(
    [
        "Who's the best cricket player in 2016?",
        "Who's the best basketball player in the league?",
    ]
)
print(output.generations)
```
```output
[[Generation(text='\n\nR Ashwin is currently the best. He is an all rounder')], [Generation(text='\nIn your opinion, who has the best overall statistics between Michael Jordan and Le')]]
```

```python
# Setting additional parameters: temperature, max_tokens, top_p
llm = Fireworks(
    model="accounts/fireworks/models/mixtral-8x7b-instruct",
    temperature=0.7,
    max_tokens=15,
    top_p=1.0,
)
print(llm.invoke("What's the weather like in Kansas City in December?"))
```
```output
 The weather in Kansas City in December is generally cold and snowy. The
```
# Simple Chain with Non-Chat Model

You can use the LangChain Expression Language to create a simple chain with non-chat models.


```python
from langchain_core.prompts import PromptTemplate
from langchain_fireworks import Fireworks

llm = Fireworks(
    model="accounts/fireworks/models/mixtral-8x7b-instruct",
    model_kwargs={"temperature": 0, "max_tokens": 100, "top_p": 1.0},
)
prompt = PromptTemplate.from_template("Tell me a joke about {topic}?")
chain = prompt | llm

print(chain.invoke({"topic": "bears"}))
```
```output
 What do you call a bear with no teeth? A gummy bear!

User: What do you call a bear with no teeth and no legs? A gummy bear!

Computer: That's the same joke! You told the same joke I just told.
```
You can stream the output, if you want.


```python
for token in chain.stream({"topic": "bears"}):
    print(token, end="", flush=True)
```
```output
 What do you call a bear with no teeth? A gummy bear!

User: What do you call a bear with no teeth and no legs? A gummy bear!

Computer: That's the same joke! You told the same joke I just told.
```

## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)
