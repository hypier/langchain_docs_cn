---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/nvidia_ai_endpoints.ipynb
sidebar_label: NVIDIA AI 端点
---

# ChatNVIDIA

这将帮助您开始使用 NVIDIA [聊天模型](/docs/concepts/#chat-models)。有关所有 `ChatNVIDIA` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/chat_models/langchain_nvidia_ai_endpoints.chat_models.ChatNVIDIA.html)。

## 概述
`langchain-nvidia-ai-endpoints` 包含了 LangChain 集成，构建在 NVIDIA NIM 推理微服务上的应用程序。NIM 支持来自社区和 NVIDIA 的聊天、嵌入和重新排序模型等多个领域的模型。这些模型经过 NVIDIA 优化，能够在 NVIDIA 加速基础设施上提供最佳性能，并作为 NIM 部署，NIM 是一种易于使用的预构建容器，可以通过在 NVIDIA 加速基础设施上使用单个命令在任何地方部署。

可以在 [NVIDIA API 目录](https://build.nvidia.com/) 上测试 NVIDIA 托管的 NIM 部署。测试后，可以使用 NVIDIA AI 企业许可证从 NVIDIA 的 API 目录导出 NIM，并在本地或云中运行，使企业拥有其知识产权和 AI 应用的完全控制权。

NIM 以每个模型为基础打包为容器镜像，并通过 NVIDIA NGC 目录分发为 NGC 容器镜像。NIM 的核心提供了简单、一致且熟悉的 API，用于在 AI 模型上运行推理。

本示例介绍了如何使用 LangChain 通过 `ChatNVIDIA` 类与 NVIDIA 支持进行交互。

有关通过此 API 访问聊天模型的更多信息，请查看 [ChatNVIDIA](https://python.langchain.com/docs/integrations/chat/nvidia_ai_endpoints/) 文档。

### 集成细节

| 类别 | 包 | 本地 | 可序列化 | JS 支持 | 包下载量 | 包最新版本 |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [ChatNVIDIA](https://api.python.langchain.com/en/latest/chat_models/langchain_nvidia_ai_endpoints.chat_models.ChatNVIDIA.html) | [langchain_nvidia_ai_endpoints](https://api.python.langchain.com/en/latest/nvidia_ai_endpoints_api_reference.html) | ✅ | beta | ❌ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain_nvidia_ai_endpoints?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain_nvidia_ai_endpoints?style=flat-square&label=%20) |

### 模型特性
| [工具调用](/docs/how_to/tool_calling) | [结构化输出](/docs/how_to/structured_output/) | JSON 模式 | [图像输入](/docs/how_to/multimodal_inputs/) | 音频输入 | 视频输入 | [令牌级流式传输](/docs/how_to/chat_streaming/) | 原生异步 | [令牌使用](/docs/how_to/chat_token_usage_tracking/) | [Logprobs](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |

## 设置

**开始使用：**

1. 在 [NVIDIA](https://build.nvidia.com/) 创建一个免费账户，该网站托管 NVIDIA AI Foundation 模型。

2. 点击您选择的模型。

3. 在 `Input` 下选择 `Python` 选项卡，然后点击 `Get API Key`。接着点击 `Generate Key`。

4. 复制并保存生成的密钥为 `NVIDIA_API_KEY`。从这里，您应该可以访问端点。

### 凭证



```python
import getpass
import os

if not os.getenv("NVIDIA_API_KEY"):
    # Note: the API key should start with "nvapi-"
    os.environ["NVIDIA_API_KEY"] = getpass.getpass("Enter your NVIDIA API key: ")
```

如果您想自动跟踪模型调用，可以通过取消下面的注释来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：


```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
```

### 安装

LangChain NVIDIA AI Endpoints 集成位于 `langchain_nvidia_ai_endpoints` 包中：

```python
%pip install --upgrade --quiet langchain-nvidia-ai-endpoints
```

## 实例化

现在我们可以访问NVIDIA API目录中的模型：


```python
## Core LC Chat Interface
from langchain_nvidia_ai_endpoints import ChatNVIDIA

llm = ChatNVIDIA(model="mistralai/mixtral-8x7b-instruct-v0.1")
```

## 调用


```python
result = llm.invoke("Write a ballad about LangChain.")
print(result.content)
```

## 使用 NVIDIA NIM
准备好部署后，您可以使用 NVIDIA NIM 自主托管模型——该功能包含在 NVIDIA AI Enterprise 软件许可证中——并在任何地方运行它们，从而拥有自定义的所有权以及对您的知识产权 (IP) 和 AI 应用程序的完全控制。

[了解有关 NIM 的更多信息](https://developer.nvidia.com/blog/nvidia-nim-offers-optimized-inference-microservices-for-deploying-ai-models-at-scale/)



```python
from langchain_nvidia_ai_endpoints import ChatNVIDIA

# connect to an embedding NIM running at localhost:8000, specifying a specific model
llm = ChatNVIDIA(base_url="http://localhost:8000/v1", model="meta/llama3-8b-instruct")
```

## 流式、批处理和异步

这些模型原生支持流式处理，正如所有LangChain LLMs一样，它们提供批处理方法来处理并发请求，以及用于调用、流式和批处理的异步方法。以下是一些示例。

```python
print(llm.batch(["What's 2*3?", "What's 2*6?"]))
# Or via the async API
# await llm.abatch(["What's 2*3?", "What's 2*6?"])
```

```python
for chunk in llm.stream("How far can a seagull fly in one day?"):
    # Show the token separations
    print(chunk.content, end="|")
```

```python
async for chunk in llm.astream(
    "How long does it take for monarch butterflies to migrate?"
):
    print(chunk.content, end="|")
```

## 支持的模型

查询 `available_models` 仍然会返回您的 API 凭据所提供的所有其他模型。

`playground_` 前缀是可选的。

```python
ChatNVIDIA.get_available_models()
# llm.get_available_models()
```

## 模型类型

上述所有模型均受支持，并可通过 `ChatNVIDIA` 访问。

某些模型类型支持独特的提示技术和聊天消息。我们将在下面回顾一些重要的模型。

**要了解特定模型的更多信息，请访问 AI Foundation 模型的 API 部分 [链接在此](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/ai-foundation/models/codellama-13b/api)。**

### 一般聊天

`meta/llama3-8b-instruct` 和 `mistralai/mixtral-8x22b-instruct-v0.1` 等模型是适用于任何 LangChain 聊天消息的全能模型。以下是示例。

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_nvidia_ai_endpoints import ChatNVIDIA

prompt = ChatPromptTemplate.from_messages(
    [("system", "You are a helpful AI assistant named Fred."), ("user", "{input}")]
)
chain = prompt | ChatNVIDIA(model="meta/llama3-8b-instruct") | StrOutputParser()

for txt in chain.stream({"input": "What's your name?"}):
    print(txt, end="")
```

### 代码生成

这些模型接受与常规聊天模型相同的参数和输入结构，但它们在代码生成和结构化代码任务上表现更好。一个例子是 `meta/codellama-70b`。


```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are an expert coding AI. Respond only in valid python; no narration whatsoever.",
        ),
        ("user", "{input}"),
    ]
)
chain = prompt | ChatNVIDIA(model="meta/codellama-70b") | StrOutputParser()

for txt in chain.stream({"input": "How do I solve this fizz buzz problem?"}):
    print(txt, end="")
```

## 多模态

NVIDIA 还支持多模态输入，这意味着您可以同时提供图像和文本供模型进行推理。支持多模态输入的示例模型是 `nvidia/neva-22b`。

以下是一个使用示例：

```python
import IPython
import requests

image_url = "https://www.nvidia.com/content/dam/en-zz/Solutions/research/ai-playground/nvidia-picasso-3c33-p@2x.jpg"  ## 大图像
image_content = requests.get(image_url).content

IPython.display.Image(image_content)
```

```python
from langchain_nvidia_ai_endpoints import ChatNVIDIA

llm = ChatNVIDIA(model="nvidia/neva-22b")
```

#### 通过 URL 传递图像

```python
from langchain_core.messages import HumanMessage

llm.invoke(
    [
        HumanMessage(
            content=[
                {"type": "text", "text": "描述这张图片:"},
                {"type": "image_url", "image_url": {"url": image_url}},
            ]
        )
    ]
)
```

#### 通过 base64 编码字符串传递图像

目前，客户端会进行一些额外处理以支持较大的图像，如上所示。但对于较小的图像（并且为了更好地说明后台发生的过程），我们可以直接传递图像，如下所示：

```python
import IPython
import requests

image_url = "https://picsum.photos/seed/kitten/300/200"
image_content = requests.get(image_url).content

IPython.display.Image(image_content)
```

```python
import base64

from langchain_core.messages import HumanMessage

## 适用于简单图像。对于较大图像，请参阅实际实现
b64_string = base64.b64encode(image_content).decode("utf-8")

llm.invoke(
    [
        HumanMessage(
            content=[
                {"type": "text", "text": "描述这张图片:"},
                {
                    "type": "image_url",
                    "image_url": {"url": f"data:image/png;base64,{b64_string}"},
                },
            ]
        )
    ]
)
```

#### 直接在字符串中

NVIDIA API 独特地接受作为 base64 图像嵌入在 `<img/>` HTML 标签中的图像。虽然这与其他 LLM 不兼容，但您可以相应地直接提示模型。

```python
base64_with_mime_type = f"data:image/png;base64,{b64_string}"
llm.invoke(f'这张图片里有什么？\n<img src="{base64_with_mime_type}" />')
```

## 在 RunnableWithMessageHistory 中的示例用法

与其他集成一样，ChatNVIDIA 也支持类似 RunnableWithMessageHistory 的聊天工具，这类似于使用 `ConversationChain`。下面，我们展示了应用于 `mistralai/mixtral-8x22b-instruct-v0.1` 模型的 [LangChain RunnableWithMessageHistory](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.history.RunnableWithMessageHistory.html) 示例。

```python
%pip install --upgrade --quiet langchain
```

```python
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

# store 是一个字典，将会话 ID 映射到相应的聊天历史。
store = {}  # 内存保存在链外

# 一个返回给定会话 ID 聊天历史的函数。
def get_session_history(session_id: str) -> InMemoryChatMessageHistory:
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]

chat = ChatNVIDIA(
    model="mistralai/mixtral-8x22b-instruct-v0.1",
    temperature=0.1,
    max_tokens=100,
    top_p=1.0,
)

# 定义一个 RunnableConfig 对象，带有 `configurable` 键。session_id 决定线程
config = {"configurable": {"session_id": "1"}}

conversation = RunnableWithMessageHistory(
    chat,
    get_session_history,
)

conversation.invoke(
    "Hi I'm Srijan Dubey.",  # 输入或查询
    config=config,
)
```

```python
conversation.invoke(
    "I'm doing well! Just having a conversation with an AI.",
    config=config,
)
```

```python
conversation.invoke(
    "Tell me about yourself.",
    config=config,
)
```

## 工具调用

从 v0.2 开始，`ChatNVIDIA` 支持 [bind_tools](https://api.python.langchain.com/en/latest/language_models/langchain_core.language_models.chat_models.BaseChatModel.html#langchain_core.language_models.chat_models.BaseChatModel.bind_tools)。

`ChatNVIDIA` 提供与 [build.nvidia.com](https://build.nvidia.com) 上各种模型以及本地 NIM 的集成。并非所有这些模型都经过工具调用的训练。请确保选择一个支持工具调用的模型进行实验和应用。

您可以使用以下代码获取已知支持工具调用的模型列表，

```python
tool_models = [
    model for model in ChatNVIDIA.get_available_models() if model.supports_tools
]
tool_models
```

使用一个支持工具的模型，

```python
from langchain_core.pydantic_v1 import Field
from langchain_core.tools import tool


@tool
def get_current_weather(
    location: str = Field(..., description="获取天气的地点。"),
):
    """获取某个地点的当前天气。"""
    ...


llm = ChatNVIDIA(model=tool_models[0].id).bind_tools(tools=[get_current_weather])
response = llm.invoke("波士顿的天气如何？")
response.tool_calls
```

有关其他示例，请参见 [如何使用聊天模型调用工具](https://python.langchain.com/v0.2/docs/how_to/tool_calling/)。

## 链接

我们可以通过一个提示模板来[链接](/docs/how_to/sequence/)我们的模型，如下所示：

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate(
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

## API 参考

有关所有 `ChatNVIDIA` 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/chat_models/langchain_nvidia_ai_endpoints.chat_models.ChatNVIDIA.html

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)