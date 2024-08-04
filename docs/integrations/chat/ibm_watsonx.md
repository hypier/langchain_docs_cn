---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/ibm_watsonx.ipynb
sidebar_label: IBM watsonx.ai
---
# ChatWatsonx

>ChatWatsonx is a wrapper for IBM [watsonx.ai](https://www.ibm.com/products/watsonx-ai) foundation models.

The aim of these examples is to show how to communicate with `watsonx.ai` models using `LangChain` LLMs API.

## Overview

### Integration details
| Class | Package | Local | Serializable | [JS support](https://js.langchain.com/v0.2/docs/integrations/chat/openai) | Package downloads | Package latest |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [ChatWatsonx](https://api.python.langchain.com/en/latest/ibm_api_reference.html) | [langchain-ibm](https://api.python.langchain.com/en/latest/ibm_api_reference.html) | ❌ | ❌ | ❌ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-ibm?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain-ibm?style=flat-square&label=%20) |

### Model features
| [Tool calling](/docs/how_to/tool_calling/) | [Structured output](/docs/how_to/structured_output/) | JSON mode | Image input | Audio input | Video input | [Token-level streaming](/docs/how_to/chat_streaming/) | Native async | [Token usage](/docs/how_to/chat_token_usage_tracking/) | [Logprobs](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ | 

## Setup

To access IBM watsonx.ai models you'll need to create an IBM watsonx.ai account, get an API key, and install the `langchain-ibm` integration package.

### Credentials

The cell below defines the credentials required to work with watsonx Foundation Model inferencing.

**Action:** Provide the IBM Cloud user API key. For details, see
[Managing user API keys](https://cloud.ibm.com/docs/account?topic=account-userapikey&interface=ui).


```python
import os
from getpass import getpass

watsonx_api_key = getpass()
os.environ["WATSONX_APIKEY"] = watsonx_api_key
```

Additionally you are able to pass additional secrets as an environment variable. 


```python
import os

os.environ["WATSONX_URL"] = "your service instance url"
os.environ["WATSONX_TOKEN"] = "your token for accessing the CPD cluster"
os.environ["WATSONX_PASSWORD"] = "your password for accessing the CPD cluster"
os.environ["WATSONX_USERNAME"] = "your username for accessing the CPD cluster"
os.environ["WATSONX_INSTANCE_ID"] = "your instance_id for accessing the CPD cluster"
```

### Installation

The LangChain IBM integration lives in the `langchain-ibm` package:


```python
!pip install -qU langchain-ibm
```

## Instantiation

You might need to adjust model `parameters` for different models or tasks. For details, refer to [Available MetaNames](https://ibm.github.io/watsonx-ai-python-sdk/fm_model.html#metanames.GenTextParamsMetaNames).


```python
parameters = {
    "decoding_method": "sample",
    "max_new_tokens": 100,
    "min_new_tokens": 1,
    "stop_sequences": ["."],
}
```

Initialize the `WatsonxLLM` class with the previously set parameters.


**Note**: 

- To provide context for the API call, you must pass the `project_id` or `space_id`. To get your project or space ID, open your project or space, go to the **Manage** tab, and click **General**. For more information see: [Project documentation](https://www.ibm.com/docs/en/watsonx-as-a-service?topic=projects) or [Deployment space documentation](https://www.ibm.com/docs/en/watsonx/saas?topic=spaces-creating-deployment).
- Depending on the region of your provisioned service instance, use one of the urls listed in [watsonx.ai API Authentication](https://ibm.github.io/watsonx-ai-python-sdk/setup_cloud.html#authentication).

In this example, we’ll use the `project_id` and Dallas URL.


You need to specify the `model_id` that will be used for inferencing. You can find the list of all the available models in [Supported foundation models](https://ibm.github.io/watsonx-ai-python-sdk/fm_model.html#ibm_watsonx_ai.foundation_models.utils.enums.ModelTypes).


```python
from langchain_ibm import ChatWatsonx

chat = ChatWatsonx(
    model_id="ibm/granite-13b-chat-v2",
    url="https://us-south.ml.cloud.ibm.com",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```

Alternatively, you can use Cloud Pak for Data credentials. For details, see [watsonx.ai software setup](https://ibm.github.io/watsonx-ai-python-sdk/setup_cpd.html).  


```python
chat = ChatWatsonx(
    model_id="ibm/granite-13b-chat-v2",
    url="PASTE YOUR URL HERE",
    username="PASTE YOUR USERNAME HERE",
    password="PASTE YOUR PASSWORD HERE",
    instance_id="openshift",
    version="4.8",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```

Instead of `model_id`, you can also pass the `deployment_id` of the previously tuned model. The entire model tuning workflow is described in [Working with TuneExperiment and PromptTuner](https://ibm.github.io/watsonx-ai-python-sdk/pt_working_with_class_and_prompt_tuner.html).


```python
chat = ChatWatsonx(
    deployment_id="PASTE YOUR DEPLOYMENT_ID HERE",
    url="https://us-south.ml.cloud.ibm.com",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```

## Invocation

To obtain completions, you can call the model directly using a string prompt.


```python
# Invocation

messages = [
    ("system", "You are a helpful assistant that translates English to French."),
    (
        "human",
        "I love you for listening to Rock.",
    ),
]

chat.invoke(messages)
```



```output
AIMessage(content="Je t'aime pour écouter la Rock.", response_metadata={'token_usage': {'generated_token_count': 12, 'input_token_count': 28}, 'model_name': 'ibm/granite-13b-chat-v2', 'system_fingerprint': '', 'finish_reason': 'stop_sequence'}, id='run-05b305ce-5401-4a10-b557-41a4b15c7f6f-0')
```



```python
# Invocation multiple chat
from langchain_core.messages import (
    HumanMessage,
    SystemMessage,
)

system_message = SystemMessage(
    content="You are a helpful assistant which telling short-info about provided topic."
)
human_message = HumanMessage(content="horse")

chat.invoke([system_message, human_message])
```



```output
AIMessage(content='Sure, I can help you with that! Horses are large, powerful mammals that belong to the family Equidae.', response_metadata={'token_usage': {'generated_token_count': 24, 'input_token_count': 24}, 'model_name': 'ibm/granite-13b-chat-v2', 'system_fingerprint': '', 'finish_reason': 'stop_sequence'}, id='run-391776ff-3b38-4768-91e8-ff64177149e5-0')
```


## Chaining
Create `ChatPromptTemplate` objects which will be responsible for creating a random question.


```python
from langchain_core.prompts import ChatPromptTemplate

system = (
    "You are a helpful assistant that translates {input_language} to {output_language}."
)
human = "{input}"
prompt = ChatPromptTemplate.from_messages([("system", system), ("human", human)])
```

Provide a inputs and run the chain.


```python
chain = prompt | chat
chain.invoke(
    {
        "input_language": "English",
        "output_language": "German",
        "input": "I love Python",
    }
)
```



```output
AIMessage(content='Ich liebe Python.', response_metadata={'token_usage': {'generated_token_count': 5, 'input_token_count': 23}, 'model_name': 'ibm/granite-13b-chat-v2', 'system_fingerprint': '', 'finish_reason': 'stop_sequence'}, id='run-1b1ccf5d-0e33-46f2-a087-e2a136ba1fb7-0')
```


## Streaming the Model output 

You can stream the model output.


```python
system_message = SystemMessage(
    content="You are a helpful assistant which telling short-info about provided topic."
)
human_message = HumanMessage(content="moon")

for chunk in chat.stream([system_message, human_message]):
    print(chunk.content, end="")
```
```output
The moon is a natural satellite of the Earth, and it has been a source of fascination for humans for centuries.
```
## Batch the Model output 

You can batch the model output.


```python
message_1 = [
    SystemMessage(
        content="You are a helpful assistant which telling short-info about provided topic."
    ),
    HumanMessage(content="cat"),
]
message_2 = [
    SystemMessage(
        content="You are a helpful assistant which telling short-info about provided topic."
    ),
    HumanMessage(content="dog"),
]

chat.batch([message_1, message_2])
```



```output
[AIMessage(content='Cats are domestic animals that belong to the Felidae family.', response_metadata={'token_usage': {'generated_token_count': 13, 'input_token_count': 24}, 'model_name': 'ibm/granite-13b-chat-v2', 'system_fingerprint': '', 'finish_reason': 'stop_sequence'}, id='run-71a8bd7a-a1aa-497b-9bdd-a4d6fe1d471a-0'),
 AIMessage(content='Dogs are domesticated mammals of the family Canidae, characterized by their adaptability to various environments and social structures.', response_metadata={'token_usage': {'generated_token_count': 24, 'input_token_count': 24}, 'model_name': 'ibm/granite-13b-chat-v2', 'system_fingerprint': '', 'finish_reason': 'stop_sequence'}, id='run-22b7a0cb-e44a-4b68-9921-872f82dcd82b-0')]
```


## Tool calling

### ChatWatsonx.bind_tools()

Please note that `ChatWatsonx.bind_tools` is on beta state, so right now we only support `mistralai/mixtral-8x7b-instruct-v01` model.

You should also redefine `max_new_tokens` parameter to get the entire model response. By default `max_new_tokens` is set to 20.


```python
from langchain_ibm import ChatWatsonx

parameters = {"max_new_tokens": 200}

chat = ChatWatsonx(
    model_id="mistralai/mixtral-8x7b-instruct-v01",
    url="https://us-south.ml.cloud.ibm.com",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```


```python
from langchain_core.pydantic_v1 import BaseModel, Field


class GetWeather(BaseModel):
    """Get the current weather in a given location"""

    location: str = Field(..., description="The city and state, e.g. San Francisco, CA")


llm_with_tools = chat.bind_tools([GetWeather])
```


```python
ai_msg = llm_with_tools.invoke(
    "Which city is hotter today: LA or NY?",
)
ai_msg
```



```output
AIMessage(content='', additional_kwargs={'function_call': {'type': 'function'}, 'tool_calls': [{'type': 'function', 'function': {'name': 'GetWeather', 'arguments': '{"location": "Los Angeles"}'}, 'id': None}, {'type': 'function', 'function': {'name': 'GetWeather', 'arguments': '{"location": "New York"}'}, 'id': None}]}, response_metadata={'token_usage': {'generated_token_count': 99, 'input_token_count': 320}, 'model_name': 'mistralai/mixtral-8x7b-instruct-v01', 'system_fingerprint': '', 'finish_reason': 'eos_token'}, id='run-38627104-f2ac-4edb-8390-d5425fb65979-0', tool_calls=[{'name': 'GetWeather', 'args': {'location': 'Los Angeles'}, 'id': None}, {'name': 'GetWeather', 'args': {'location': 'New York'}, 'id': None}])
```


### AIMessage.tool_calls
Notice that the AIMessage has a `tool_calls` attribute. This contains in a standardized ToolCall format that is model-provider agnostic.


```python
ai_msg.tool_calls
```



```output
[{'name': 'GetWeather', 'args': {'location': 'Los Angeles'}, 'id': None},
 {'name': 'GetWeather', 'args': {'location': 'New York'}, 'id': None}]
```


## API reference

For detailed documentation of all IBM watsonx.ai features and configurations head to the API reference: https://api.python.langchain.com/en/latest/ibm_api_reference.html


## Related

- Chat model [conceptual guide](/docs/concepts/#chat-models)
- Chat model [how-to guides](/docs/how_to/#chat-models)
