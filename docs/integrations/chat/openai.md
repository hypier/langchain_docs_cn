---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/openai.ipynb
sidebar_label: OpenAI
---
# ChatOpenAI

This notebook provides a quick overview for getting started with OpenAI [chat models](/docs/concepts/#chat-models). For detailed documentation of all ChatOpenAI features and configurations head to the [API reference](https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.base.ChatOpenAI.html).

OpenAI has several chat models. You can find information about their latest models and their costs, context windows, and supported input types in the [OpenAI docs](https://platform.openai.com/docs/models).

:::info Azure OpenAI

Note that certain OpenAI models can also be accessed via the [Microsoft Azure platform](https://azure.microsoft.com/en-us/products/ai-services/openai-service). To use the Azure OpenAI service use the [AzureChatOpenAI integration](/docs/integrations/chat/azure_chat_openai/).

:::

## Overview

### Integration details
| Class | Package | Local | Serializable | [JS support](https://js.langchain.com/v0.2/docs/integrations/chat/openai) | Package downloads | Package latest |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [ChatOpenAI](https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.base.ChatOpenAI.html) | [langchain-openai](https://api.python.langchain.com/en/latest/openai_api_reference.html) | ❌ | beta | ✅ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-openai?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain-openai?style=flat-square&label=%20) |

### Model features
| [Tool calling](/docs/how_to/tool_calling) | [Structured output](/docs/how_to/structured_output/) | JSON mode | Image input | Audio input | Video input | [Token-level streaming](/docs/how_to/chat_streaming/) | Native async | [Token usage](/docs/how_to/chat_token_usage_tracking/) | [Logprobs](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ | 

## Setup

To access OpenAI models you'll need to create an OpenAI account, get an API key, and install the `langchain-openai` integration package.

### Credentials

Head to https://platform.openai.com to sign up to OpenAI and generate an API key. Once you've done this set the OPENAI_API_KEY environment variable:


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("Enter your OpenAI API key: ")
```
```output
Enter your OpenAI API key:  ········
```
If you want to get automated tracing of your model calls you can also set your [LangSmith](https://docs.smith.langchain.com/) API key by uncommenting below:


```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### Installation

The LangChain OpenAI integration lives in the `langchain-openai` package:


```python
%pip install -qU langchain-openai
```

## Instantiation

Now we can instantiate our model object and generate chat completions:


```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4o",
    temperature=0,
    max_tokens=None,
    timeout=None,
    max_retries=2,
    # api_key="...",  # if you prefer to pass api key in directly instaed of using env vars
    # base_url="...",
    # organization="...",
    # other params...
)
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
AIMessage(content="J'adore la programmation.", response_metadata={'token_usage': {'completion_tokens': 5, 'prompt_tokens': 31, 'total_tokens': 36}, 'model_name': 'gpt-4o', 'system_fingerprint': 'fp_43dfabdef1', 'finish_reason': 'stop', 'logprobs': None}, id='run-012cffe2-5d3d-424d-83b5-51c6d4a593d1-0', usage_metadata={'input_tokens': 31, 'output_tokens': 5, 'total_tokens': 36})
```



```python
print(ai_msg.content)
```
```output
J'adore la programmation.
```
## Chaining

We can [chain](/docs/how_to/sequence/) our model with a prompt template like so:


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
AIMessage(content='Ich liebe Programmieren.', response_metadata={'token_usage': {'completion_tokens': 5, 'prompt_tokens': 26, 'total_tokens': 31}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': 'fp_b28b39ffa8', 'finish_reason': 'stop', 'logprobs': None}, id='run-94fa6741-c99b-4513-afce-c3f562631c79-0')
```


## Tool calling

OpenAI has a [tool calling](https://platform.openai.com/docs/guides/function-calling) (we use "tool calling" and "function calling" interchangeably here) API that lets you describe tools and their arguments, and have the model return a JSON object with a tool to invoke and the inputs to that tool. tool-calling is extremely useful for building tool-using chains and agents, and for getting structured outputs from models more generally.

### ChatOpenAI.bind_tools()

With `ChatOpenAI.bind_tools`, we can easily pass in Pydantic classes, dict schemas, LangChain tools, or even functions as tools to the model. Under the hood these are converted to an OpenAI tool schemas, which looks like:
```
{
    "name": "...",
    "description": "...",
    "parameters": {...}  # JSONSchema
}
```
and passed in every model invocation.


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
AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_H7fABDuzEau48T10Qn0Lsh0D', 'function': {'arguments': '{"location":"San Francisco"}', 'name': 'GetWeather'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 15, 'prompt_tokens': 70, 'total_tokens': 85}, 'model_name': 'gpt-3.5-turbo-0125', 'system_fingerprint': 'fp_b28b39ffa8', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-b469135e-2718-446a-8164-eef37e672ba2-0', tool_calls=[{'name': 'GetWeather', 'args': {'location': 'San Francisco'}, 'id': 'call_H7fABDuzEau48T10Qn0Lsh0D'}])
```


### AIMessage.tool_calls
Notice that the AIMessage has a `tool_calls` attribute. This contains in a standardized ToolCall format that is model-provider agnostic.


```python
ai_msg.tool_calls
```



```output
[{'name': 'GetWeather',
  'args': {'location': 'San Francisco'},
  'id': 'call_H7fABDuzEau48T10Qn0Lsh0D'}]
```


For more on binding tools and tool call outputs, head to the [tool calling](/docs/how_to/function_calling) docs.

## Fine-tuning

You can call fine-tuned OpenAI models by passing in your corresponding `modelName` parameter.

This generally takes the form of `ft:{OPENAI_MODEL_NAME}:{ORG_NAME}::{MODEL_ID}`. For example:


```python
fine_tuned_model = ChatOpenAI(
    temperature=0, model_name="ft:gpt-3.5-turbo-0613:langchain::7qTVM5AR"
)

fine_tuned_model(messages)
```



```output
AIMessage(content="J'adore la programmation.", additional_kwargs={}, example=False)
```


## API reference

For detailed documentation of all ChatOpenAI features and configurations head to the API reference: https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.base.ChatOpenAI.html


## Related

- Chat model [conceptual guide](/docs/concepts/#chat-models)
- Chat model [how-to guides](/docs/how_to/#chat-models)
