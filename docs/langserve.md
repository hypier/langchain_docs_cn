---
custom_edit_url:
---

# 🦜️🏓 LangServe

[![Release Notes](https://img.shields.io/github/release/langchain-ai/langserve)](https://github.com/langchain-ai/langserve/releases)
[![Downloads](https://static.pepy.tech/badge/langserve/month)](https://pepy.tech/project/langserve)
[![Open Issues](https://img.shields.io/github/issues-raw/langchain-ai/langserve)](https://github.com/langchain-ai/langserve/issues)
[![](https://dcbadge.vercel.app/api/server/6adMQxSpJS?compact=true&style=flat)](https://discord.com/channels/1038097195422978059/1170024642245832774)

## 概述

[LangServe](https://github.com/langchain-ai/langserve) 帮助开发者将 `LangChain` [可运行对象和链](https://python.langchain.com/docs/expression_language/) 部署为 REST API。

该库与 [FastAPI](https://fastapi.tiangolo.com/) 集成，并使用 [pydantic](https://docs.pydantic.dev/latest/) 进行数据验证。

此外，它还提供了一个客户端，可以用来调用部署在服务器上的可运行对象。
JavaScript 客户端可在 [LangChain.js](https://js.langchain.com/docs/ecosystem/langserve) 中找到。

## 特性

- 从您的 LangChain 对象自动推断输入和输出模式，并在每个 API 调用中强制执行，提供丰富的错误信息
- 带有 JSONSchema 和 Swagger 的 API 文档页面（插入示例链接）
- 高效的 `/invoke`、`/batch` 和 `/stream` 端点，支持单个服务器上的多个并发请求
- `/stream_log` 端点用于流式传输您的链/代理中的所有（或部分）中间步骤
- **新** 在 0.0.40 中，支持 `/stream_events`，使流式传输更简单，无需解析 `/stream_log` 的输出。
- `/playground/` 页面，具有流式输出和中间步骤
- 内置（可选）追踪到 [LangSmith](https://www.langchain.com/langsmith)，只需添加您的 API 密钥（请参见 [说明](https://docs.smith.langchain.com/)）
- 所有功能均基于经过实战检验的开源 Python 库，如 FastAPI、Pydantic、uvloop 和 asyncio。
- 使用客户端 SDK 调用 LangServe 服务器，就像它是一个本地运行的 Runnable（或直接调用 HTTP API）
- [LangServe Hub](https://github.com/langchain-ai/langchain/blob/master/templates/README.md)

## ⚠️ LangGraph 兼容性

LangServe 主要用于部署简单的 Runnables，并与 langchain-core 中的知名原语一起使用。

如果您需要 LangGraph 的部署选项，您应该考虑 [LangGraph Cloud (beta)](https://langchain-ai.github.io/langgraph/cloud/)，它更适合部署 LangGraph 应用程序。

## 限制

- 目前不支持来自服务器的事件的客户端回调
- 使用 Pydantic V2 时将不会生成 OpenAPI 文档。Fast API 不支持 [混合使用 pydantic v1 和 v2 命名空间](https://github.com/tiangolo/fastapi/issues/10360)。有关更多详细信息，请参见下面的部分。

## 安全性

- 版本 0.0.13 - 0.0.15 中的漏洞 -- playground 端点允许访问服务器上的任意文件。 [在 0.0.16 中解决](https://github.com/langchain-ai/langserve/pull/98)。

## 安装

对于客户端和服务器：

```bash
pip install "langserve[all]"
```

或者使用 `pip install "langserve[client]"` 安装客户端代码，使用 `pip install "langserve[server]"` 安装服务器代码。

## LangChain CLI 🛠️

使用 `LangChain` CLI 快速启动 `LangServe` 项目。

要使用 langchain CLI，请确保您已安装最新版本的 `langchain-cli`。您可以通过 `pip install -U langchain-cli` 安装它。

## 设置

**注意**：我们使用 `poetry` 进行依赖管理。请参考 poetry [文档](https://python-poetry.org/docs/) 以了解更多信息。

### 1. 使用 langchain cli 命令创建新应用

```sh
langchain app new my-app
```

### 2. 在 add_routes 中定义可运行的内容。前往 server.py 并进行编辑

```sh
add_routes(app. NotImplemented)
```

### 3. 使用 `poetry` 添加第三方包（例如，langchain-openai、langchain-anthropic、langchain-mistral 等）。

```sh
poetry add [package-name] // e.g `poetry add langchain-openai`
```

### 4. 设置相关环境变量。例如，

```sh
export OPENAI_API_KEY="sk-..."
```

### 5. 服务你的应用

```sh
poetry run langchain serve --port=8100
```

## 示例

快速启动您的 LangServe 实例，使用 [LangChain 模板](https://github.com/langchain-ai/langchain/blob/master/templates/README.md)。

有关更多示例，请查看模板 [索引](https://github.com/langchain-ai/langchain/blob/master/templates/docs/INDEX.md) 或 [示例](https://github.com/langchain-ai/langserve/tree/main/examples) 目录。

| 描述                                                                                                                                                                                                                                                       | 链接                                                                                                                                                                                                                                   |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LLMs** 最小示例，保留 OpenAI 和 Anthropic 聊天模型。使用异步，支持批处理和流式传输。                                                                                                                                                                       | [server](https://github.com/langchain-ai/langserve/tree/main/examples/llm/server.py), [client](https://github.com/langchain-ai/langserve/blob/main/examples/llm/client.ipynb)                                                       |
| **Retriever** 简单服务器，暴露可运行的检索器。                                                                                                                                                                                                            | [server](https://github.com/langchain-ai/langserve/tree/main/examples/retrieval/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/retrieval/client.ipynb)                                           |
| **Conversational Retriever** 通过 LangServe 暴露的 [Conversational Retriever](https://python.langchain.com/docs/expression_language/cookbook/retrieval#conversational-retrieval-chain)                                                             | [server](https://github.com/langchain-ai/langserve/tree/main/examples/conversational_retrieval_chain/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/conversational_retrieval_chain/client.ipynb) |
| 基于 [OpenAI tools](https://python.langchain.com/docs/modules/agents/agent_types/openai_functions_agent) 的 **Agent**，没有 **对话历史**。                                                                                                                  | [server](https://github.com/langchain-ai/langserve/tree/main/examples/agent/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/agent/client.ipynb)                                                   |
| 基于 [OpenAI tools](https://python.langchain.com/docs/modules/agents/agent_types/openai_functions_agent) 的 **Agent**，带有 **对话历史**。                                                                                                                  | [server](https://github.com/langchain-ai/langserve/blob/main/examples/agent_with_history/server.py), [client](https://github.com/langchain-ai/langserve/blob/main/examples/agent_with_history/client.ipynb)                         |
| [RunnableWithMessageHistory](https://python.langchain.com/docs/expression_language/how_to/message_history) 实现后端持久化聊天，通过客户端提供的 `session_id` 进行键控。                                                                                     | [server](https://github.com/langchain-ai/langserve/tree/main/examples/chat_with_persistence/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/chat_with_persistence/client.ipynb)                   |
| [RunnableWithMessageHistory](https://python.langchain.com/docs/expression_language/how_to/message_history) 实现后端持久化聊天，通过客户端提供的 `conversation_id` 和 `user_id` 进行键控（有关正确实现 `user_id` 的信息，请参见 Auth）。                | [server](https://github.com/langchain-ai/langserve/tree/main/examples/chat_with_persistence_and_user/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/chat_with_persistence_and_user/client.ipynb) |
| [Configurable Runnable](https://python.langchain.com/docs/expression_language/how_to/configure) 创建一个支持运行时配置索引名称的检索器。                                                                                                                   | [server](https://github.com/langchain-ai/langserve/tree/main/examples/configurable_retrieval/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/configurable_retrieval/client.ipynb)                 |
| [Configurable Runnable](https://python.langchain.com/docs/expression_language/how_to/configure) 显示可配置字段和可配置替代项。                                                                                                                                 | [server](https://github.com/langchain-ai/langserve/tree/main/examples/configurable_chain/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/configurable_chain/client.ipynb)                         |
| **APIHandler** 显示如何使用 `APIHandler` 代替 `add_routes`。这为开发人员定义端点提供了更大的灵活性。与所有 FastAPI 模式配合良好，但需要更多的努力。                                                                                                   | [server](https://github.com/langchain-ai/langserve/tree/main/examples/api_handler_examples/server.py)                                                                                                                                   |
| **LCEL 示例** 使用 LCEL 操作字典输入的示例。                                                                                                                                                                                                               | [server](https://github.com/langchain-ai/langserve/tree/main/examples/passthrough_dict/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/passthrough_dict/client.ipynb)                             |
| **Auth** 使用 `add_routes`：可以应用于与应用程序关联的所有端点的简单身份验证。（单独使用对实现每个用户逻辑没有用。）                                                                                                                               | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/global_deps/server.py)                                                                                                                                   |
| **Auth** 使用 `add_routes`：基于路径依赖关系的简单身份验证机制。（单独使用对实现每个用户逻辑没有用。）                                                                                                                                              | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/path_dependencies/server.py)                                                                                                                             |
| **Auth** 使用 `add_routes`：实现每个用户逻辑和身份验证，适用于使用每个请求配置修改器的端点。（**注意**：目前不与 OpenAPI 文档集成。）                                                                                                               | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/per_req_config_modifier/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/auth/per_req_config_modifier/client.ipynb)     |
| **Auth** 使用 `APIHandler`：实现每个用户逻辑和身份验证，显示如何仅在用户拥有的文档中进行搜索。                                                                                                                                                    | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/api_handler/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/auth/api_handler/client.ipynb)                             |
| **Widgets** 可用于游乐场（文件上传和聊天）的不同小部件。                                                                                                                                                                                                    | [server](https://github.com/langchain-ai/langserve/tree/main/examples/widgets/chat/tuples/server.py)                                                                                                                                |
| **Widgets** 用于 LangServe 游乐场的文件上传小部件。                                                                                                                                                                                                        | [server](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing/client.ipynb)                               |

## 示例应用程序

### 服务器

这是一个部署 OpenAI 聊天模型、Anthropic 聊天模型以及一个使用 Anthropic 模型讲述关于某个主题笑话的链的服务器。

```python
#!/usr/bin/env python
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatAnthropic, ChatOpenAI
from langserve import add_routes

app = FastAPI(
    title="LangChain Server",
    version="1.0",
    description="A simple api server using Langchain's Runnable interfaces",
)

add_routes(
    app,
    ChatOpenAI(model="gpt-3.5-turbo-0125"),
    path="/openai",
)

add_routes(
    app,
    ChatAnthropic(model="claude-3-haiku-20240307"),
    path="/anthropic",
)

model = ChatAnthropic(model="claude-3-haiku-20240307")
prompt = ChatPromptTemplate.from_template("tell me a joke about {topic}")
add_routes(
    app,
    prompt | model,
    path="/joke",
)

if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="localhost", port=8000)
```

如果您打算从浏览器调用您的端点，您还需要设置 CORS 头。您可以使用 FastAPI 的内置中间件来实现：

```python
from fastapi.middleware.cors import CORSMiddleware

# Set all CORS enabled origins
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
    expose_headers=["*"],
)
```

### 文档

如果您已经部署了上述服务器，可以通过以下方式查看生成的 OpenAPI 文档：

> ⚠️ 如果使用 pydantic v2，_invoke_、_batch_、_stream_、_stream_log_ 的文档将不会生成。有关更多详细信息，请参见下面的 [Pydantic](#pydantic) 部分。

```sh
curl localhost:8000/docs
```

确保 **添加** `/docs` 后缀。

> ⚠️ 索引页面 `/` 没有按 **设计** 定义，因此 `curl localhost:8000` 或访问该 URL 将返回 404。如果您希望在 `/` 上有内容，请定义一个端点 `@app.get("/")`。

### 客户端

Python SDK

```python

from langchain.schema import SystemMessage, HumanMessage
from langchain.prompts import ChatPromptTemplate
from langchain.schema.runnable import RunnableMap
from langserve import RemoteRunnable

openai = RemoteRunnable("http://localhost:8000/openai/")
anthropic = RemoteRunnable("http://localhost:8000/anthropic/")
joke_chain = RemoteRunnable("http://localhost:8000/joke/")

joke_chain.invoke({"topic": "parrots"})

# 或者异步
await joke_chain.ainvoke({"topic": "parrots"})

prompt = [
    SystemMessage(content='表现得像一只猫或一只鹦鹉。'),
    HumanMessage(content='你好！')
]

# 支持astream
async for msg in anthropic.astream(prompt):
    print(msg, end="", flush=True)

prompt = ChatPromptTemplate.from_messages(
    [("system", "给我讲一个关于{topic}的长故事")]
)

# 可以定义自定义链
chain = prompt | RunnableMap({
    "openai": openai,
    "anthropic": anthropic,
})

chain.batch([{"topic": "parrots"}, {"topic": "cats"}])
```

在 TypeScript 中（需要 LangChain.js 版本 0.0.166 或更高）：

```typescript
import { RemoteRunnable } from "@langchain/core/runnables/remote";

const chain = new RemoteRunnable({
  url: `http://localhost:8000/joke/`,
});
const result = await chain.invoke({
  topic: "cats",
});
```

使用 `requests` 的 Python：

```python
import requests

response = requests.post(
    "http://localhost:8000/joke/invoke",
    json={'input': {'topic': 'cats'}}
)
response.json()
```

您也可以使用 `curl`：

```sh
curl --location --request POST 'http://localhost:8000/joke/invoke' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "input": {
            "topic": "cats"
        }
    }'
```

## 端点

以下代码：

```python
...
add_routes(
    app,
    runnable,
    path="/my_runnable",
)
```

将这些端点添加到服务器：

- `POST /my_runnable/invoke` - 在单个输入上调用可运行对象
- `POST /my_runnable/batch` - 在一批输入上调用可运行对象
- `POST /my_runnable/stream` - 在单个输入上调用并流式传输输出
- `POST /my_runnable/stream_log` - 在单个输入上调用并流式传输输出，包括生成的中间步骤输出
- `POST /my_runnable/astream_events` - 在单个输入上调用并流式传输生成的事件，包括来自中间步骤的事件。
- `GET /my_runnable/input_schema` - 可运行对象输入的 json schema
- `GET /my_runnable/output_schema` - 可运行对象输出的 json schema
- `GET /my_runnable/config_schema` - 可运行对象配置的 json schema

这些端点与
[LangChain 表达语言接口](https://python.langchain.com/docs/expression_language/interface) 匹配——有关更多详细信息，请参考此文档。

## Playground

您可以在 `/my_runnable/playground/` 找到可运行的 playground 页面。这
提供了一个简单的 UI
来 [配置](https://python.langchain.com/docs/expression_language/how_to/configure)
并调用您的可运行程序，支持流式输出和中间步骤。

<p align="center">
<img src="https://github.com/langchain-ai/langserve/assets/3205522/5ca56e29-f1bb-40f4-84b5-15916384a276" width="50%"/>
</p>

### 小部件

游乐场支持 [小部件](#playground-widgets)，可以用于测试您的可运行程序与不同的输入。有关更多详细信息，请参见下面的 [小部件](#widgets) 部分。

### 共享

此外，对于可配置的可运行项，游乐场将允许您配置可运行项并分享带有该配置的链接：

<p align="center">
<img src="https://github.com/langchain-ai/langserve/assets/3205522/86ce9c59-f8e4-4d08-9fa3-62030e0f521d" width="50%"/>
</p>

## 聊天游乐场

LangServe 还支持一个以聊天为中心的游乐场，可以在 `/my_runnable/playground/` 下选择并使用。与一般的游乐场不同，仅支持某些类型的可运行项 - 可运行项的输入模式必须是一个 `dict`，其格式为：

- 单个键，该键的值必须是一个聊天消息列表。
- 两个键，一个键的值是消息列表，另一个键表示最新的消息。

我们建议您使用第一种格式。

可运行项还必须返回 `AIMessage` 或字符串。

要启用此功能，您必须在添加路由时设置 `playground_type="chat",`。以下是一个示例：

```python
# Declare a chain
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful, professional assistant named Cob."),
        MessagesPlaceholder(variable_name="messages"),
    ]
)

chain = prompt | ChatAnthropic(model="claude-2")


class InputChat(BaseModel):
    """Input for the chat endpoint."""

    messages: List[Union[HumanMessage, AIMessage, SystemMessage]] = Field(
        ...,
        description="The chat messages representing the current conversation.",
    )


add_routes(
    app,
    chain.with_types(input_type=InputChat),
    enable_feedback_endpoint=True,
    enable_public_trace_link_endpoint=True,
    playground_type="chat",
)
```

如果您使用 LangSmith，您还可以在路由上设置 `enable_feedback_endpoint=True` 以启用每条消息后的点赞/点踩按钮，并设置 `enable_public_trace_link_endpoint=True` 以添加一个按钮，用于创建运行的公共跟踪。请注意，您还需要设置以下环境变量：

```bash
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_PROJECT="YOUR_PROJECT_NAME"
export LANGCHAIN_API_KEY="YOUR_API_KEY"
```

以下是启用上述两个选项的示例：

<p align="center">
<img src="./.github/img/chat_playground.png" width="50%"/>
</p>

注意：如果您启用公共跟踪链接，您的链的内部结构将被暴露。我们建议仅在演示或测试时使用此设置。

## 传统链

LangServe 同时支持 Runnables（通过 [LangChain 表达式语言](https://python.langchain.com/docs/expression_language/) 构建）和传统链（继承自 `Chain`）。然而，某些传统链的输入模式可能不完整或不正确，导致错误。这可以通过更新 LangChain 中那些链的 `input_schema` 属性来修复。如果您遇到任何错误，请在此仓库中提交问题，我们将努力解决。

## 部署

### 部署到 AWS

您可以使用 [AWS Copilot CLI](https://aws.github.io/copilot-cli/) 部署到 AWS

```bash
copilot init --app [application-name] --name [service-name] --type 'Load Balanced Web Service' --dockerfile './Dockerfile' --deploy
```

点击 [这里](https://aws.amazon.com/containers/copilot/) 了解更多信息。

### 部署到 Azure

您可以使用 Azure Container Apps（无服务器）部署到 Azure：

```
az containerapp up --name [container-app-name] --source . --resource-group [resource-group-name] --environment  [environment-name] --ingress external --target-port 8001 --env-vars=OPENAI_API_KEY=your_key
```

您可以在 [这里](https://learn.microsoft.com/en-us/azure/container-apps/containerapp-up) 找到更多信息。

### 部署到 GCP

您可以使用以下命令部署到 GCP Cloud Run：

```
gcloud run deploy [your-service-name] --source . --port 8001 --allow-unauthenticated --region us-central1 --set-env-vars=OPENAI_API_KEY=your_key
```

### 社区贡献

#### 部署到 Railway

[示例 Railway 仓库](https://github.com/PaulLockett/LangServe-Railway/tree/main)

[![在 Railway 上部署](https://railway.app/button.svg)](https://railway.app/template/pW9tXP?referralCode=c-aq4K)

## Pydantic

LangServe 对 Pydantic 2 提供支持，但有一些限制。

1. 在使用 Pydantic V2 时，invoke/batch/stream/stream_log 的 OpenAPI 文档将不会生成。Fast API 不支持 [混合使用 pydantic v1 和 v2 命名空间]。要解决此问题，请使用 `pip install pydantic==1.10.17`。
2. LangChain 在 Pydantic v2 中使用 v1 命名空间。请阅读
   [以下指南以确保与 LangChain 的兼容性](https://github.com/langchain-ai/langchain/discussions/9337)

除了这些限制之外，我们预计 API 端点、游乐场和其他任何功能都能正常工作。

## 高级

### 处理身份验证

如果您需要为服务器添加身份验证，请阅读 Fast API 的文档关于 [依赖项](https://fastapi.tiangolo.com/tutorial/dependencies/) 和 [安全性](https://fastapi.tiangolo.com/tutorial/security/)。

下面的示例展示了如何使用 FastAPI 原语将身份验证逻辑连接到 LangServe 端点。

您需要提供实际的身份验证逻辑、用户表等。

如果您不确定自己在做什么，可以尝试使用现有的解决方案 [Auth0](https://auth0.com/)。

#### 使用 add_routes

如果您使用 `add_routes`，请查看 [这里](https://github.com/langchain-ai/langserve/tree/main/examples/auth) 的示例。

| 描述                                                                                                                                                                               | 链接                                                                                                                                                                                                 |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Auth** 使用 `add_routes`：简单的身份验证，可以应用于与应用程序关联的所有端点。（单独使用对实现每个用户逻辑没有用。）                                                             | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/global_deps/server.py)                                                                                                   |
| **Auth** 使用 `add_routes`：基于路径依赖的简单身份验证机制。（单独使用对实现每个用户逻辑没有用。）                                                                            | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/path_dependencies/server.py)                                                                                             |
| **Auth** 使用 `add_routes`：实现每个用户逻辑和身份验证，用于使用每个请求配置修改器的端点。（**注意**：目前不与 OpenAPI 文档集成。）                                               | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/per_req_config_modifier/server.py)，[client](https://github.com/langchain-ai/langserve/tree/main/examples/auth/per_req_config_modifier/client.ipynb) |

另外，您可以使用 FastAPI 的 [中间件](https://fastapi.tiangolo.com/tutorial/middleware/)。

使用全局依赖和路径依赖的优点在于身份验证将在 OpenAPI 文档页面中得到正确支持，但这些不足以实现每个用户的逻辑（例如，创建一个只能在用户拥有的文档中搜索的应用程序）。

如果您需要实现每个用户的逻辑，可以使用 `per_req_config_modifier` 或 `APIHandler`（如下）来实现此逻辑。

**每个用户**

如果您需要授权或依赖于用户的逻辑，请在使用 `add_routes` 时指定 `per_req_config_modifier`。使用一个可调用对象接收原始 `Request` 对象，并可以从中提取相关信息以用于身份验证和授权目的。

#### 使用 APIHandler

如果您对 FastAPI 和 Python 感到舒适，可以使用 LangServe 的 [APIHandler](https://github.com/langchain-ai/langserve/blob/main/examples/api_handler_examples/server.py)。

| 描述                                                                                                                                                                                                 | 链接                                                                                                                                                                                                   |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Auth** 使用 `APIHandler`：实现每个用户逻辑和身份验证，展示如何仅在用户拥有的文档中进行搜索。                                                                                                   | [server](https://github.com/langchain-ai/langserve/tree/main/examples/auth/api_handler/server.py)，[client](https://github.com/langchain-ai/langserve/tree/main/examples/auth/api_handler/client.ipynb) |
| **APIHandler** 展示如何使用 `APIHandler` 而不是 `add_routes`。这为开发人员定义端点提供了更多灵活性。与所有 FastAPI 模式配合良好，但需要更多的努力。                                            | [server](https://github.com/langchain-ai/langserve/tree/main/examples/api_handler_examples/server.py)，[client](https://github.com/langchain-ai/langserve/tree/main/examples/api_handler_examples/client.ipynb) |

这需要更多的工作，但可以让您完全控制端点定义，因此您可以根据需要进行自定义逻辑处理。

### 文件

LLM 应用程序通常处理文件。可以实现文件处理的不同架构；从高层次来看：

1. 文件可以通过专用端点上传到服务器，并使用单独的端点进行处理
2. 文件可以通过值（文件的字节）或引用（例如，指向文件内容的 s3 url）进行上传
3. 处理端点可以是阻塞的或非阻塞的
4. 如果需要大量处理，处理可以转移到专用进程池

您应该确定适合您应用程序的架构。

目前，要通过值上传文件到可运行的实例，请使用 base64 编码文件（`multipart/form-data` 目前不支持）。

这是一个 [示例](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing)，展示了如何使用 base64 编码将文件发送到远程可运行实例。

请记住，您始终可以通过引用（例如，s3 url）上传文件，或将它们作为 multipart/form-data 上传到专用端点。

### 自定义输入和输出类型

输入和输出类型在所有可运行对象上定义。

您可以通过 `input_schema` 和 `output_schema` 属性访问它们。

`LangServe` 使用这些类型进行验证和文档编制。

如果您想覆盖默认推断的类型，可以使用 `with_types` 方法。

以下是一个玩具示例来说明这个概念：

```python
from typing import Any

from fastapi import FastAPI
from langchain.schema.runnable import RunnableLambda

app = FastAPI()


def func(x: Any) -> int:
    """Mistyped function that should accept an int but accepts anything."""
    return x + 1


runnable = RunnableLambda(func).with_types(
    input_type=int,
)

add_routes(app, runnable)
```

### 自定义用户类型

如果您希望数据反序列化为 pydantic 模型而不是等效的字典表示，请从 `CustomUserType` 继承。

目前，这种类型仅在 _服务器_ 端工作，用于指定所需的 _解码_ 行为。如果从此类型继承，服务器将保持解码类型为 pydantic 模型，而不是将其转换为字典。

```python
from fastapi import FastAPI
from langchain.schema.runnable import RunnableLambda

from langserve import add_routes
from langserve.schema import CustomUserType

app = FastAPI()


class Foo(CustomUserType):
    bar: int


def func(foo: Foo) -> int:
    """示例函数，期望 Foo 类型为 pydantic 模型"""
    assert isinstance(foo, Foo)
    return foo.bar


# 注意，输入和输出类型会自动推断！
# 您不需要指定它们。
# runnable = RunnableLambda(func).with_types( # <-- 在这种情况下不需要
#     input_type=Foo,
#     output_type=int,
#
add_routes(app, RunnableLambda(func), path="/foo")
```

### Playground Widgets

该游乐场允许您从后端定义可运行的自定义小部件。

以下是一些示例：

| 描述                                                                                 | 链接                                                                                                                                                                                                  |
| :---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Widgets** 可以与游乐场一起使用的不同小部件（文件上传和聊天）                     | [server](https://github.com/langchain-ai/langserve/tree/main/examples/widgets/chat/tuples/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/widgets/client.ipynb)     |
| **Widgets** 用于LangServe游乐场的文件上传小部件。                                   | [server](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing/server.py), [client](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing/client.ipynb) |

#### Schema

- 小部件在字段级别上指定，并作为输入类型的JSON架构的一部分进行传输
- 小部件必须包含一个名为`type`的键，其值为已知小部件列表中的一个
- 其他小部件键将与描述JSON对象中路径的值相关联

```typescript
type JsonPath = number | string | (number | string)[];
type NameSpacedPath = { title: string; path: JsonPath }; // Using title to mimick json schema, but can use namespace
type OneOfPath = { oneOf: JsonPath[] };

type Widget = {
  type: string; // Some well known type (e.g., base64file, chat etc.)
  [key: string]: JsonPath | NameSpacedPath | OneOfPath;
};
```

### 可用小部件

目前用户只能手动指定两个小部件：

1. 文件上传小部件
2. 聊天记录小部件

请参见下面关于这些小部件的更多信息。

在游乐场 UI 上的所有其他小部件都是根据 Runnable 的配置模式自动创建和管理的。当您创建可配置的 Runnable 时，游乐场应该为您创建适当的小部件以控制行为。

#### 文件上传小部件

允许在 UI 游乐场中创建一个文件上传输入，用于上传为 base64 编码字符串的文件。以下是完整的 [示例](https://github.com/langchain-ai/langserve/tree/main/examples/file_processing)。

代码片段：

```python
try:
    from pydantic.v1 import Field
except ImportError:
    from pydantic import Field

from langserve import CustomUserType


# ATTENTION: Inherit from CustomUserType instead of BaseModel otherwise
#            the server will decode it into a dict instead of a pydantic model.
class FileProcessingRequest(CustomUserType):
    """Request including a base64 encoded file."""

    # The extra field is used to specify a widget for the playground UI.
    file: str = Field(..., extra={"widget": {"type": "base64file"}})
    num_chars: int = 100

```

示例小部件：

<p align="center">
<img src="https://github.com/langchain-ai/langserve/assets/3205522/52199e46-9464-4c2e-8be8-222250e08c3f" width="50%"/>
</p>

### 聊天小部件

查看 [小部件示例](https://github.com/langchain-ai/langserve/tree/main/examples/widgets/chat/tuples/server.py)。

要定义一个聊天小部件，请确保传递 "type": "chat"。

- "input" 是 _Request_ 中包含新输入消息的字段的 JSONPath。
- "output" 是 _Response_ 中包含新输出消息的字段的 JSONPath。
- 如果整个输入或输出应按原样使用，则不需要指定这些字段（例如，如果输出是聊天消息的列表）。

以下是一个代码片段：

```python
class ChatHistory(CustomUserType):
    chat_history: List[Tuple[str, str]] = Field(
        ...,
        examples=[[("human input", "ai response")]],
        extra={"widget": {"type": "chat", "input": "question", "output": "answer"}},
    )
    question: str


def _format_to_messages(input: ChatHistory) -> List[BaseMessage]:
    """Format the input to a list of messages."""
    history = input.chat_history
    user_input = input.question

    messages = []

    for human, ai in history:
        messages.append(HumanMessage(content=human))
        messages.append(AIMessage(content=ai))
    messages.append(HumanMessage(content=user_input))
    return messages


model = ChatOpenAI()
chat_model = RunnableParallel({"answer": (RunnableLambda(_format_to_messages) | model)})
add_routes(
    app,
    chat_model.with_types(input_type=ChatHistory),
    config_keys=["configurable"],
    path="/chat",
)
```

示例小部件：

<p align="center">
<img src="https://github.com/langchain-ai/langserve/assets/3205522/a71ff37b-a6a9-4857-a376-cf27c41d3ca4" width="50%"/>
</p>

您还可以直接将消息列表作为参数指定，如下所示：

```python
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assisstant named Cob."),
        MessagesPlaceholder(variable_name="messages"),
    ]
)

chain = prompt | ChatAnthropic(model="claude-2")


class MessageListInput(BaseModel):
    """Input for the chat endpoint."""
    messages: List[Union[HumanMessage, AIMessage]] = Field(
        ...,
        description="The chat messages representing the current conversation.",
        extra={"widget": {"type": "chat", "input": "messages"}},
    )


add_routes(
    app,
    chain.with_types(input_type=MessageListInput),
    path="/chat",
)
```

请参见 [此示例文件](https://github.com/langchain-ai/langserve/tree/main/examples/widgets/chat/message_list/server.py) 以获取示例。

### 启用 / 禁用端点 (LangServe >=0.0.33)

您可以在为给定链添加路由时启用 / 禁用暴露的端点。

如果您希望在将 langserve 升级到新版本时确保不会获取新的端点，请使用 `enabled_endpoints`。

启用：下面的代码将仅启用 `invoke`、`batch` 及其对应的 `config_hash` 端点变体。

```python
add_routes(app, chain, enabled_endpoints=["invoke", "batch", "config_hashes"], path="/mychain")
```

禁用：下面的代码将禁用该链的 playground。

```python
add_routes(app, chain, disabled_endpoints=["playground"], path="/mychain")
```