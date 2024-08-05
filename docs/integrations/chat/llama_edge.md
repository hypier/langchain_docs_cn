---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/llama_edge.ipynb
---

# LlamaEdge

[LlamaEdge](https://github.com/second-state/LlamaEdge) 允许您与 [GGUF](https://github.com/ggerganov/llama.cpp/blob/master/gguf-py/README.md) 格式的 LLM 进行本地聊天和通过聊天服务进行交互。

- `LlamaEdgeChatService` 为开发者提供一个与 OpenAI API 兼容的服务，通过 HTTP 请求与 LLM 进行聊天。

- `LlamaEdgeChatLocal` 使开发者能够在本地与 LLM 进行聊天（即将推出）。

`LlamaEdgeChatService` 和 `LlamaEdgeChatLocal` 都运行在由 [WasmEdge Runtime](https://wasmedge.org/) 驱动的基础设施上，该基础设施为 LLM 推理任务提供轻量级和可移植的 WebAssembly 容器环境。

## 通过 API 服务聊天

`LlamaEdgeChatService` 在 `llama-api-server` 上运行。按照 [llama-api-server 快速入门](https://github.com/second-state/llama-utils/tree/main/api-server#readme) 中的步骤，您可以托管自己的 API 服务，这样您就可以在任何设备上与您喜欢的模型进行聊天，只要有互联网连接。

```python
from langchain_community.chat_models.llama_edge import LlamaEdgeChatService
from langchain_core.messages import HumanMessage, SystemMessage
```

### 在非流模式下与LLMs聊天


```python
# service url
service_url = "https://b008-54-186-154-209.ngrok-free.app"

# create wasm-chat service instance
chat = LlamaEdgeChatService(service_url=service_url)

# create message sequence
system_message = SystemMessage(content="You are an AI assistant")
user_message = HumanMessage(content="What is the capital of France?")
messages = [system_message, user_message]

# chat with wasm-chat service
response = chat.invoke(messages)

print(f"[Bot] {response.content}")
```
```output
[Bot] Hello! The capital of France is Paris.
```

### 与 LLMs 进行流式对话


```python
# service url
service_url = "https://b008-54-186-154-209.ngrok-free.app"

# create wasm-chat service instance
chat = LlamaEdgeChatService(service_url=service_url, streaming=True)

# create message sequence
system_message = SystemMessage(content="You are an AI assistant")
user_message = HumanMessage(content="What is the capital of Norway?")
messages = [
    system_message,
    user_message,
]

output = ""
for chunk in chat.stream(messages):
    # print(chunk.content, end="", flush=True)
    output += chunk.content

print(f"[Bot] {output}")
```
```output
[Bot]   Hello! I'm happy to help you with your question. The capital of Norway is Oslo.
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)