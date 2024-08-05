---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/minimax.ipynb
sidebar_label: MiniMax
---

# MiniMaxChat

[Minimax](https://api.minimax.chat) 是一家提供LLM服务的中国初创公司，面向企业和个人。

本示例介绍如何使用LangChain与MiniMax推理进行聊天交互。


```python
import os

os.environ["MINIMAX_GROUP_ID"] = "MINIMAX_GROUP_ID"
os.environ["MINIMAX_API_KEY"] = "MINIMAX_API_KEY"
```


```python
from langchain_community.chat_models import MiniMaxChat
from langchain_core.messages import HumanMessage
```


```python
chat = MiniMaxChat()
```


```python
chat(
    [
        HumanMessage(
            content="Translate this sentence from English to French. I love programming."
        )
    ]
)
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)