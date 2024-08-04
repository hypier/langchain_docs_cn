---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/tongyi.ipynb
sidebar_label: Tongyi Qwen
---
# ChatTongyi
Tongyi Qwen is a large language model developed by Alibaba's Damo Academy. It is capable of understanding user intent through natural language understanding and semantic analysis, based on user input in natural language. It provides services and assistance to users in different domains and tasks. By providing clear and detailed instructions, you can obtain results that better align with your expectations.
In this notebook, we will introduce how to use langchain with [Tongyi](https://www.aliyun.com/product/dashscope) mainly in `Chat` corresponding
 to the package `langchain/chat_models` in langchain


```python
# Install the package
%pip install --upgrade --quiet  dashscope
```
```output
Note: you may need to restart the kernel to use updated packages.
```

```python
# Get a new token: https://help.aliyun.com/document_detail/611472.html?spm=a2c4g.2399481.0.0
from getpass import getpass

DASHSCOPE_API_KEY = getpass()
```


```python
import os

os.environ["DASHSCOPE_API_KEY"] = DASHSCOPE_API_KEY
```


```python
from langchain_community.chat_models.tongyi import ChatTongyi
from langchain_core.messages import HumanMessage

chatLLM = ChatTongyi(
    streaming=True,
)
res = chatLLM.stream([HumanMessage(content="hi")], streaming=True)
for r in res:
    print("chat resp:", r)
```
```output
chat resp: content='Hello' id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
chat resp: content='!' id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
chat resp: content=' How' id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
chat resp: content=' can I assist you today' id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
chat resp: content='?' id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
chat resp: content='' response_metadata={'finish_reason': 'stop', 'request_id': '921db2c5-4d53-9a89-8e87-e4ad6a671237', 'token_usage': {'input_tokens': 20, 'output_tokens': 9, 'total_tokens': 29}} id='run-f2301962-6d46-423c-8afa-1e667bd11e2b'
```

```python
from langchain_core.messages import HumanMessage, SystemMessage

messages = [
    SystemMessage(
        content="You are a helpful assistant that translates English to French."
    ),
    HumanMessage(
        content="Translate this sentence from English to French. I love programming."
    ),
]
chatLLM(messages)
```
```output
/Users/cheese/PARA/Projects/langchain-contribution/langchain/libs/core/langchain_core/_api/deprecation.py:119: LangChainDeprecationWarning: The method `BaseChatModel.__call__` was deprecated in langchain-core 0.1.7 and will be removed in 0.2.0. Use invoke instead.
  warn_deprecated(
```


```output
AIMessage(content="J'adore programmer.", response_metadata={'model_name': 'qwen-turbo', 'finish_reason': 'stop', 'request_id': 'ae725086-0ffa-9728-8c72-b204c7bc7eeb', 'token_usage': {'input_tokens': 36, 'output_tokens': 6, 'total_tokens': 42}}, id='run-060cc103-ef5f-4c8a-af40-792ac7f40c26-0')
```


## Tool Calling
ChatTongyi supports tool calling API that lets you describe tools and their arguments, and have the model return a JSON object with a tool to invoke and the inputs to that tool.

### Use with `bind_tools`


```python
from langchain_community.chat_models.tongyi import ChatTongyi
from langchain_core.tools import tool


@tool
def multiply(first_int: int, second_int: int) -> int:
    """Multiply two integers together."""
    return first_int * second_int


llm = ChatTongyi(model="qwen-turbo")

llm_with_tools = llm.bind_tools([multiply])

msg = llm_with_tools.invoke("What's 5 times forty two")

print(msg)
```
```output
content='' additional_kwargs={'tool_calls': [{'function': {'name': 'multiply', 'arguments': '{"first_int": 5, "second_int": 42}'}, 'id': '', 'type': 'function'}]} response_metadata={'model_name': 'qwen-turbo', 'finish_reason': 'tool_calls', 'request_id': '4acf0e36-44af-987a-a0c0-8b5c5eaa1a8b', 'token_usage': {'input_tokens': 200, 'output_tokens': 25, 'total_tokens': 225}} id='run-0ecd0f09-1d20-4e55-a4f3-f14d1f710ae7-0' tool_calls=[{'name': 'multiply', 'args': {'first_int': 5, 'second_int': 42}, 'id': ''}]
```
### Construct args manually


```python
from langchain_community.chat_models.tongyi import ChatTongyi
from langchain_core.messages import HumanMessage, SystemMessage

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_current_time",
            "description": "当你想知道现在的时间时非常有用。",
            "parameters": {},
        },
    },
    {
        "type": "function",
        "function": {
            "name": "get_current_weather",
            "description": "当你想查询指定城市的天气时非常有用。",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "城市或县区，比如北京市、杭州市、余杭区等。",
                    }
                },
            },
            "required": ["location"],
        },
    },
]

messages = [
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="What is the weather like in San Francisco?"),
]
chatLLM = ChatTongyi()
llm_kwargs = {"tools": tools, "result_format": "message"}
ai_message = chatLLM.bind(**llm_kwargs).invoke(messages)
ai_message
```



```output
AIMessage(content='', additional_kwargs={'tool_calls': [{'function': {'name': 'get_current_weather', 'arguments': '{"location": "San Francisco"}'}, 'id': '', 'type': 'function'}]}, response_metadata={'model_name': 'qwen-turbo', 'finish_reason': 'tool_calls', 'request_id': '87ef33d2-5c6b-9457-91e2-39faad7120eb', 'token_usage': {'input_tokens': 229, 'output_tokens': 19, 'total_tokens': 248}}, id='run-7939ba7f-e3f7-46f8-980b-30499b52723c-0', tool_calls=[{'name': 'get_current_weather', 'args': {'location': 'San Francisco'}, 'id': ''}])
```


## Tongyi With Vision
Qwen-VL(qwen-vl-plus/qwen-vl-max) are models that can process images.


```python
from langchain_community.chat_models import ChatTongyi
from langchain_core.messages import HumanMessage

chatLLM = ChatTongyi(model_name="qwen-vl-max")
image_message = {
    "image": "https://lilianweng.github.io/posts/2023-06-23-agent/agent-overview.png",
}
text_message = {
    "text": "summarize this picture",
}
message = HumanMessage(content=[text_message, image_message])
chatLLM.invoke([message])
```



```output
AIMessage(content=[{'text': 'The image presents a flowchart of an artificial intelligence system. The system is divided into two main components: short-term memory and long-term memory, which are connected to the "Memory" box.\n\nFrom the "Memory" box, there are three branches leading to different functionalities:\n\n1. "Tools" - This branch represents various tools that the AI system can utilize, including "Calendar()", "Calculator()", "CodeInterpreter()", "Search()" and others not explicitly listed.\n\n2. "Action" - This branch represents the action taken by the AI system based on its processing of information. It\'s connected to both the "Tools" and the "Agent" boxes.\n\n3. "Planning" - This branch represents the planning process of the AI system, which involves reflection, self-critics, chain of thoughts, subgoal decomposition, and other processes not shown.\n\nThe central component of the system is the "Agent" box, which seems to orchestrate the flow of information between the different components. The "Agent" interacts with the "Tools" and "Memory" boxes, suggesting it plays a crucial role in the AI\'s decision-making process. \n\nOverall, the image depicts a complex and interconnected artificial intelligence system, where different components work together to process information, make decisions, and take actions.'}], response_metadata={'model_name': 'qwen-vl-max', 'finish_reason': 'stop', 'request_id': '6a2b9e90-7c3b-960d-8a10-6a0cf9991ae5', 'token_usage': {'input_tokens': 1262, 'output_tokens': 260, 'image_tokens': 1232}}, id='run-fd030661-c734-4580-b977-b77d42680742-0')
```



## Related

- Chat model [conceptual guide](/docs/concepts/#chat-models)
- Chat model [how-to guides](/docs/how_to/#chat-models)