---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/zhipuai.ipynb
sidebar_label: ZHIPU AI
---

# ZHIPU AI

本笔记本展示了如何在 LangChain 中使用 [ZHIPU AI API](https://open.bigmodel.cn/dev/api) 和 langchain.chat_models.ChatZhipuAI。

>[*GLM-4*](https://open.bigmodel.cn/) 是一个与人类意图对齐的多语言大型语言模型，具备问答、多轮对话和代码生成的能力。新一代基础模型 GLM-4 的整体性能相比于前一代有了显著提升，支持更长的上下文；更强的多模态能力；支持更快的推理速度和更高的并发性，大幅降低推理成本；同时，GLM-4 增强了智能体的能力。

## 开始使用

### 安装
首先，确保在您的 Python 环境中安装了 zhipuai 包。运行以下命令：

```python
#!pip install --upgrade httpx httpx-sse PyJWT
```

### 导入所需模块
安装后，将必要的模块导入到您的 Python 脚本中：


```python
from langchain_community.chat_models import ChatZhipuAI
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage
```

### 设置您的 API 密钥
登录 [ZHIPU AI](https://open.bigmodel.cn/login?redirect=%2Fusercenter%2Fapikeys) 获取 API 密钥以访问我们的模型。

```python
import os

os.environ["ZHIPUAI_API_KEY"] = "zhipuai_api_key"
```

### 初始化 ZHIPU AI 聊天模型
以下是初始化聊天模型的方法：


```python
chat = ChatZhipuAI(
    model="glm-4",
    temperature=0.5,
)
```

### 基本用法
使用系统和人类消息调用模型，如下所示：


```python
messages = [
    AIMessage(content="Hi."),
    SystemMessage(content="Your role is a poet."),
    HumanMessage(content="Write a short poem about AI in four lines."),
]
```


```python
response = chat.invoke(messages)
print(response.content)  # Displays the AI-generated poem
```

## 高级功能

### 流媒体支持
要实现持续交互，请使用流媒体功能：

```python
from langchain_core.callbacks.manager import CallbackManager
from langchain_core.callbacks.streaming_stdout import StreamingStdOutCallbackHandler
```

```python
streaming_chat = ChatZhipuAI(
    model="glm-4",
    temperature=0.5,
    streaming=True,
    callback_manager=CallbackManager([StreamingStdOutCallbackHandler()]),
)
```

```python
streaming_chat(messages)
```

### 异步调用
对于非阻塞调用，请使用异步方法：


```python
async_chat = ChatZhipuAI(
    model="glm-4",
    temperature=0.5,
)
```


```python
response = await async_chat.agenerate([messages])
print(response)
```

### 使用函数调用

GLM-4 模型可以与函数调用一起使用，使用以下代码运行一个简单的 LangChain json_chat_agent。

```python
os.environ["TAVILY_API_KEY"] = "tavily_api_key"
```

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_json_chat_agent
from langchain_community.tools.tavily_search import TavilySearchResults

tools = [TavilySearchResults(max_results=1)]
prompt = hub.pull("hwchase17/react-chat-json")
llm = ChatZhipuAI(temperature=0.01, model="glm-4")

agent = create_json_chat_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent, tools=tools, verbose=True, handle_parsing_errors=True
)
```

```python
agent_executor.invoke({"input": "what is LangChain?"})
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)