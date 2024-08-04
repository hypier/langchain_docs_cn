---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/chat_models_universal_init.ipynb
---
# How to init any model in one line

Many LLM applications let end users specify what model provider and model they want the application to be powered by. This requires writing some logic to initialize different ChatModels based on some user configuration. The `init_chat_model()` helper method makes it easy to initialize a number of different model integrations without having to worry about import paths and class names.

:::tip Supported models

See the [init_chat_model()](https://api.python.langchain.com/en/latest/chat_models/langchain.chat_models.base.init_chat_model.html) API reference for a full list of supported integrations.

Make sure you have the integration packages installed for any model providers you want to support. E.g. you should have `langchain-openai` installed to init an OpenAI model.

:::

:::info Requires ``langchain >= 0.2.8``

This functionality was added in ``langchain-core == 0.2.8``. Please make sure your package is up to date.

:::


```python
%pip install -qU langchain>=0.2.8 langchain-openai langchain-anthropic langchain-google-vertexai
```

## Basic usage


```python
from langchain.chat_models import init_chat_model

# Returns a langchain_openai.ChatOpenAI instance.
gpt_4o = init_chat_model("gpt-4o", model_provider="openai", temperature=0)
# Returns a langchain_anthropic.ChatAnthropic instance.
claude_opus = init_chat_model(
    "claude-3-opus-20240229", model_provider="anthropic", temperature=0
)
# Returns a langchain_google_vertexai.ChatVertexAI instance.
gemini_15 = init_chat_model(
    "gemini-1.5-pro", model_provider="google_vertexai", temperature=0
)

# Since all model integrations implement the ChatModel interface, you can use them in the same way.
print("GPT-4o: " + gpt_4o.invoke("what's your name").content + "\n")
print("Claude Opus: " + claude_opus.invoke("what's your name").content + "\n")
print("Gemini 1.5: " + gemini_15.invoke("what's your name").content + "\n")
```
```output
GPT-4o: I'm an AI created by OpenAI, and I don't have a personal name. You can call me Assistant! How can I help you today?

Claude Opus: My name is Claude. It's nice to meet you!

Gemini 1.5: I am a large language model, trained by Google. I do not have a name.
```
## Inferring model provider

For common and distinct model names `init_chat_model()` will attempt to infer the model provider. See the [API reference](https://api.python.langchain.com/en/latest/chat_models/langchain.chat_models.base.init_chat_model.html) for a full list of inference behavior. E.g. any model that starts with `gpt-3...` or `gpt-4...` will be inferred as using model provider `openai`.


```python
gpt_4o = init_chat_model("gpt-4o", temperature=0)
claude_opus = init_chat_model("claude-3-opus-20240229", temperature=0)
gemini_15 = init_chat_model("gemini-1.5-pro", temperature=0)
```

## Creating a configurable model

You can also create a runtime-configurable model by specifying `configurable_fields`. If you don't specify a `model` value, then "model" and "model_provider" be configurable by default.


```python
configurable_model = init_chat_model(temperature=0)

configurable_model.invoke(
    "what's your name", config={"configurable": {"model": "gpt-4o"}}
)
```



```output
AIMessage(content="I'm an AI language model created by OpenAI, and I don't have a personal name. You can call me Assistant or any other name you prefer! How can I assist you today?", response_metadata={'token_usage': {'completion_tokens': 37, 'prompt_tokens': 11, 'total_tokens': 48}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_d576307f90', 'finish_reason': 'stop', 'logprobs': None}, id='run-5428ab5c-b5c0-46de-9946-5d4ca40dbdc8-0', usage_metadata={'input_tokens': 11, 'output_tokens': 37, 'total_tokens': 48})
```



```python
configurable_model.invoke(
    "what's your name", config={"configurable": {"model": "claude-3-5-sonnet-20240620"}}
)
```



```output
AIMessage(content="My name is Claude. It's nice to meet you!", response_metadata={'id': 'msg_012XvotUJ3kGLXJUWKBVxJUi', 'model': 'claude-3-5-sonnet-20240620', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 11, 'output_tokens': 15}}, id='run-1ad1eefe-f1c6-4244-8bc6-90e2cb7ee554-0', usage_metadata={'input_tokens': 11, 'output_tokens': 15, 'total_tokens': 26})
```


### Configurable model with default values

We can create a configurable model with default model values, specify which parameters are configurable, and add prefixes to configurable params:


```python
first_llm = init_chat_model(
    model="gpt-4o",
    temperature=0,
    configurable_fields=("model", "model_provider", "temperature", "max_tokens"),
    config_prefix="first",  # useful when you have a chain with multiple models
)

first_llm.invoke("what's your name")
```



```output
AIMessage(content="I'm an AI language model created by OpenAI, and I don't have a personal name. You can call me Assistant or any other name you prefer! How can I assist you today?", response_metadata={'token_usage': {'completion_tokens': 37, 'prompt_tokens': 11, 'total_tokens': 48}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_ce0793330f', 'finish_reason': 'stop', 'logprobs': None}, id='run-3923e328-7715-4cd6-b215-98e4b6bf7c9d-0', usage_metadata={'input_tokens': 11, 'output_tokens': 37, 'total_tokens': 48})
```



```python
first_llm.invoke(
    "what's your name",
    config={
        "configurable": {
            "first_model": "claude-3-5-sonnet-20240620",
            "first_temperature": 0.5,
            "first_max_tokens": 100,
        }
    },
)
```



```output
AIMessage(content="My name is Claude. It's nice to meet you!", response_metadata={'id': 'msg_01RyYR64DoMPNCfHeNnroMXm', 'model': 'claude-3-5-sonnet-20240620', 'stop_reason': 'end_turn', 'stop_sequence': None, 'usage': {'input_tokens': 11, 'output_tokens': 15}}, id='run-22446159-3723-43e6-88df-b84797e7751d-0', usage_metadata={'input_tokens': 11, 'output_tokens': 15, 'total_tokens': 26})
```


### Using a configurable model declaratively

We can call declarative operations like `bind_tools`, `with_structured_output`, `with_configurable`, etc. on a configurable model and chain a configurable model in the same way that we would a regularly instantiated chat model object.


```python
from langchain_core.pydantic_v1 import BaseModel, Field


class GetWeather(BaseModel):
    """Get the current weather in a given location"""

    location: str = Field(..., description="The city and state, e.g. San Francisco, CA")


class GetPopulation(BaseModel):
    """Get the current population in a given location"""

    location: str = Field(..., description="The city and state, e.g. San Francisco, CA")


llm = init_chat_model(temperature=0)
llm_with_tools = llm.bind_tools([GetWeather, GetPopulation])

llm_with_tools.invoke(
    "what's bigger in 2024 LA or NYC", config={"configurable": {"model": "gpt-4o"}}
).tool_calls
```



```output
[{'name': 'GetPopulation',
  'args': {'location': 'Los Angeles, CA'},
  'id': 'call_sYT3PFMufHGWJD32Hi2CTNUP'},
 {'name': 'GetPopulation',
  'args': {'location': 'New York, NY'},
  'id': 'call_j1qjhxRnD3ffQmRyqjlI1Lnk'}]
```



```python
llm_with_tools.invoke(
    "what's bigger in 2024 LA or NYC",
    config={"configurable": {"model": "claude-3-5-sonnet-20240620"}},
).tool_calls
```



```output
[{'name': 'GetPopulation',
  'args': {'location': 'Los Angeles, CA'},
  'id': 'toolu_01CxEHxKtVbLBrvzFS7GQ5xR'},
 {'name': 'GetPopulation',
  'args': {'location': 'New York City, NY'},
  'id': 'toolu_013A79qt5toWSsKunFBDZd5S'}]
```

