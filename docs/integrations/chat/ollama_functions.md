---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/ollama_functions.ipynb
sidebar_label: Ollama Functions
sidebar_class_name: hidden
---
# OllamaFunctions

:::warning

This was an experimental wrapper that attempts to bolt-on tool calling support to models that do not natively support it. The [primary Ollama integration](/docs/integrations/chat/ollama/) now supports tool calling, and should be used instead.

:::
This notebook shows how to use an experimental wrapper around Ollama that gives it [tool calling capabilities](https://python.langchain.com/v0.2/docs/concepts/#functiontool-calling).

Note that more powerful and capable models will perform better with complex schema and/or multiple functions. The examples below use llama3 and phi3 models.
For a complete list of supported models and model variants, see the [Ollama model library](https://ollama.ai/library).

## Overview

### Integration details

|                                                                Class                                                                | Package | Local | Serializable | JS support | Package downloads | Package latest |
|:-----------------------------------------------------------------------------------------------------------------------------------:|:-------:|:-----:|:------------:|:----------:|:-----------------:|:--------------:|
| [OllamaFunctions](https://api.python.langchain.com/en/latest/llms/langchain_experimental.llms.ollama_function.OllamaFunctions.html) | [langchain-experimental](https://api.python.langchain.com/en/latest/openai_api_reference.html) | ✅ | ❌ | ❌ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-experimental?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain-experimental?style=flat-square&label=%20) |

### Model features

| [Tool calling](/docs/how_to/tool_calling/) | [Structured output](/docs/how_to/structured_output/) | JSON mode | Image input | Audio input | Video input | [Token-level streaming](/docs/how_to/chat_streaming/) | Native async | [Token usage](/docs/how_to/chat_token_usage_tracking/) | [Logprobs](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ |

## Setup

To access `OllamaFunctions` you will need to install `langchain-experimental` integration package.
Follow [these instructions](https://github.com/jmorganca/ollama) to set up and run a local Ollama instance as well as download and serve [supported models](https://ollama.com/library).

### Credentials

Credentials support is not present at this time.

### Installation

The `OllamaFunctions` class lives in the `langchain-experimental` package:



```python
%pip install -qU langchain-experimental
```

## Instantiation

`OllamaFunctions` takes the same init parameters as `ChatOllama`. 

In order to use tool calling, you must also specify `format="json"`.


```python
from langchain_experimental.llms.ollama_functions import OllamaFunctions

llm = OllamaFunctions(model="phi3")
```

## Invocation


```python
messages = [
    (
        "system",
        "You are a helpful assistant that translates English to French. Translate the user sentence.",
    ),
    ("human", "I love programming."),
]
ai_msg = llm.invoke(messages)
ai_msg
```



```output
AIMessage(content="J'adore programmer.", id='run-94815fcf-ae11-438a-ba3f-00819328b5cd-0')
```



```python
ai_msg.content
```



```output
"J'adore programmer."
```


## Chaining

We can [chain](https://python.langchain.com/v0.2/docs/how_to/sequence/) our model with a prompt template like so:


```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant that translates {input_language} to {output_language}.",
        ),
        ("human", "{input}"),
    ]
)

chain = prompt | llm
chain.invoke(
    {
        "input_language": "English",
        "output_language": "German",
        "input": "I love programming.",
    }
)
```



```output
AIMessage(content='Programmieren ist sehr verrückt! Es freut mich, dass Sie auf Programmierung so positiv eingestellt sind.', id='run-ee99be5e-4d48-4ab6-b602-35415f0bdbde-0')
```


## Tool Calling

### OllamaFunctions.bind_tools()

With `OllamaFunctions.bind_tools`, we can easily pass in Pydantic classes, dict schemas, LangChain tools, or even functions as tools to the model. Under the hood these are converted to a tool definition schemas, which looks like:


```python
from langchain_core.pydantic_v1 import BaseModel, Field


class GetWeather(BaseModel):
    """Get the current weather in a given location"""

    location: str = Field(..., description="The city and state, e.g. San Francisco, CA")


llm_with_tools = llm.bind_tools([GetWeather])
```


```python
ai_msg = llm_with_tools.invoke(
    "what is the weather like in San Francisco",
)
ai_msg
```



```output
AIMessage(content='', id='run-b9769435-ec6a-4cb8-8545-5a5035fc19bd-0', tool_calls=[{'name': 'GetWeather', 'args': {'location': 'San Francisco, CA'}, 'id': 'call_064c4e1cb27e4adb9e4e7ed60362ecc9'}])
```


### AIMessage.tool_calls

Notice that the AIMessage has a `tool_calls` attribute. This contains in a standardized `ToolCall` format that is model-provider agnostic.


```python
ai_msg.tool_calls
```



```output
[{'name': 'GetWeather',
  'args': {'location': 'San Francisco, CA'},
  'id': 'call_064c4e1cb27e4adb9e4e7ed60362ecc9'}]
```


For more on binding tools and tool call outputs, head to the [tool calling](../../how_to/function_calling.md) docs.

## API reference

For detailed documentation of all ToolCallingLLM features and configurations head to the API reference: https://api.python.langchain.com/en/latest/llms/langchain_experimental.llms.ollama_functions.OllamaFunctions.html



## Related

- Chat model [conceptual guide](/docs/concepts/#chat-models)
- Chat model [how-to guides](/docs/how_to/#chat-models)
