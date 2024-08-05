---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/upstage.ipynb
sidebar_label: Upstage
---

# ChatUpstage

本笔记本介绍如何开始使用 Upstage 聊天模型。

## 安装

安装 `langchain-upstage` 包。

```bash
pip install -U langchain-upstage
```

## 环境设置

确保设置以下环境变量：

- `UPSTAGE_API_KEY`: 您来自 [Upstage 控制台](https://console.upstage.ai/) 的 Upstage API 密钥。

## 用法


```python
import os

os.environ["UPSTAGE_API_KEY"] = "YOUR_API_KEY"
```


```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_upstage import ChatUpstage

chat = ChatUpstage()
```


```python
# 使用聊天调用
chat.invoke("Hello, how are you?")
```


```python
# 使用聊天流
for m in chat.stream("Hello, how are you?"):
    print(m)
```

## 链接


```python
# using chain
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful assistant that translates English to French."),
        ("human", "Translate this sentence from English to French. {english_text}."),
    ]
)
chain = prompt | chat

chain.invoke({"english_text": "Hello, how are you?"})
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)