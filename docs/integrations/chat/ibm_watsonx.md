---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/ibm_watsonx.ipynb
sidebar_label: IBM watsonx.ai
---

# ChatWatsonx

>ChatWatsonx 是 IBM [watsonx.ai](https://www.ibm.com/products/watsonx-ai) 基础模型的封装。

这些示例的目的是展示如何使用 `LangChain` LLMs API 与 `watsonx.ai` 模型进行通信。

## 概述

### 集成细节
| 类别 | 包 | 本地 | 可序列化 | [JS 支持](https://js.langchain.com/v0.2/docs/integrations/chat/openai) | 包下载量 | 包最新版本 |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [ChatWatsonx](https://api.python.langchain.com/en/latest/ibm_api_reference.html) | [langchain-ibm](https://api.python.langchain.com/en/latest/ibm_api_reference.html) | ❌ | ❌ | ❌ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-ibm?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain-ibm?style=flat-square&label=%20) |

### 模型特性
| [工具调用](/docs/how_to/tool_calling/) | [结构化输出](/docs/how_to/structured_output/) | JSON模式 | 图像输入 | 音频输入 | 视频输入 | [令牌级流式传输](/docs/how_to/chat_streaming/) | 原生异步 | [令牌使用](/docs/how_to/chat_token_usage_tracking/) | [对数概率](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ |

## 设置

要访问 IBM watsonx.ai 模型，您需要创建一个 IBM watsonx.ai 账户，获取 API 密钥，并安装 `langchain-ibm` 集成包。

### 凭证

下面的单元格定义了与 watsonx Foundation Model 推理相关的凭证要求。

**操作：** 提供 IBM Cloud 用户 API 密钥。有关详细信息，请参见
[管理用户 API 密钥](https://cloud.ibm.com/docs/account?topic=account-userapikey&interface=ui)。

```python
import os
from getpass import getpass

watsonx_api_key = getpass()
os.environ["WATSONX_APIKEY"] = watsonx_api_key
```

此外，您还可以作为环境变量传递其他机密。

```python
import os

os.environ["WATSONX_URL"] = "your service instance url"
os.environ["WATSONX_TOKEN"] = "your token for accessing the CPD cluster"
os.environ["WATSONX_PASSWORD"] = "your password for accessing the CPD cluster"
os.environ["WATSONX_USERNAME"] = "your username for accessing the CPD cluster"
os.environ["WATSONX_INSTANCE_ID"] = "your instance_id for accessing the CPD cluster"
```

### 安装

LangChain IBM 集成位于 `langchain-ibm` 包中：


```python
!pip install -qU langchain-ibm
```

## 实例化

您可能需要调整不同模型或任务的 `parameters`。有关详细信息，请参阅 [可用的 MetaNames](https://ibm.github.io/watsonx-ai-python-sdk/fm_model.html#metanames.GenTextParamsMetaNames)。

```python
parameters = {
    "decoding_method": "sample",
    "max_new_tokens": 100,
    "min_new_tokens": 1,
    "stop_sequences": ["."],
}
```

使用之前设置的参数初始化 `WatsonxLLM` 类。

**注意**：

- 为了提供 API 调用的上下文，您必须传递 `project_id` 或 `space_id`。要获取您的项目或空间 ID，请打开您的项目或空间，转到 **管理** 标签，然后点击 **常规**。有关更多信息，请参见：[项目文档](https://www.ibm.com/docs/en/watsonx-as-a-service?topic=projects) 或 [部署空间文档](https://www.ibm.com/docs/en/watsonx/saas?topic=spaces-creating-deployment)。
- 根据您配置的服务实例的区域，使用 [watsonx.ai API 认证](https://ibm.github.io/watsonx-ai-python-sdk/setup_cloud.html#authentication) 中列出的 URL 之一。

在这个例子中，我们将使用 `project_id` 和达拉斯 URL。

您需要指定将用于推理的 `model_id`。您可以在 [支持的基础模型](https://ibm.github.io/watsonx-ai-python-sdk/fm_model.html#ibm_watsonx_ai.foundation_models.utils.enums.ModelTypes) 中找到所有可用模型的列表。

```python
from langchain_ibm import ChatWatsonx

chat = ChatWatsonx(
    model_id="ibm/granite-13b-chat-v2",
    url="https://us-south.ml.cloud.ibm.com",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```

或者，您可以使用 Cloud Pak for Data 凭据。有关详细信息，请参见 [watsonx.ai 软件设置](https://ibm.github.io/watsonx-ai-python-sdk/setup_cpd.html)。

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

除了 `model_id`，您还可以传递之前调整过的模型的 `deployment_id`。整个模型调整工作流程描述在 [与 TuneExperiment 和 PromptTuner 一起工作](https://ibm.github.io/watsonx-ai-python-sdk/pt_working_with_class_and_prompt_tuner.html)。

```python
chat = ChatWatsonx(
    deployment_id="PASTE YOUR DEPLOYMENT_ID HERE",
    url="https://us-south.ml.cloud.ibm.com",
    project_id="PASTE YOUR PROJECT_ID HERE",
    params=parameters,
)
```

## 调用

要获取补全，您可以直接使用字符串提示调用模型。

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

## 链接
创建 `ChatPromptTemplate` 对象，负责生成随机问题。

```python
from langchain_core.prompts import ChatPromptTemplate

system = (
    "You are a helpful assistant that translates {input_language} to {output_language}."
)
human = "{input}"
prompt = ChatPromptTemplate.from_messages([("system", system), ("human", human)])
```

提供输入并运行链。

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

## 流式输出模型结果

您可以流式输出模型结果。

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

## 批量模型输出

您可以批量处理模型输出。

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

## 工具调用

### ChatWatsonx.bind_tools()

请注意，`ChatWatsonx.bind_tools` 处于测试状态，因此目前我们仅支持 `mistralai/mixtral-8x7b-instruct-v01` 模型。

您还应该重新定义 `max_new_tokens` 参数，以获取整个模型的响应。默认情况下，`max_new_tokens` 设置为 20。


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
    """获取给定地点的当前天气"""

    location: str = Field(..., description="城市和州，例如：旧金山，加州")


llm_with_tools = chat.bind_tools([GetWeather])
```


```python
ai_msg = llm_with_tools.invoke(
    "今天哪个城市更热：洛杉矶还是纽约？",
)
ai_msg
```



```output
AIMessage(content='', additional_kwargs={'function_call': {'type': 'function'}, 'tool_calls': [{'type': 'function', 'function': {'name': 'GetWeather', 'arguments': '{"location": "Los Angeles"}'}, 'id': None}, {'type': 'function', 'function': {'name': 'GetWeather', 'arguments': '{"location": "New York"}'}, 'id': None}]}, response_metadata={'token_usage': {'generated_token_count': 99, 'input_token_count': 320}, 'model_name': 'mistralai/mixtral-8x7b-instruct-v01', 'system_fingerprint': '', 'finish_reason': 'eos_token'}, id='run-38627104-f2ac-4edb-8390-d5425fb65979-0', tool_calls=[{'name': 'GetWeather', 'args': {'location': 'Los Angeles'}, 'id': None}, {'name': 'GetWeather', 'args': {'location': 'New York'}, 'id': None}])
```

### AIMessage.tool_calls
请注意，AIMessage具有`tool_calls`属性。它包含以标准化的ToolCall格式表示的信息，与模型提供者无关。

```python
ai_msg.tool_calls
```

```output
[{'name': 'GetWeather', 'args': {'location': 'Los Angeles'}, 'id': None},
 {'name': 'GetWeather', 'args': {'location': 'New York'}, 'id': None}]
```

## API 参考

有关所有 IBM watsonx.ai 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/ibm_api_reference.html

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)