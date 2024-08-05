---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/ernie.ipynb
sidebar_label: Ernie Bot 聊天
---

# ErnieBotChat

[ERNIE-Bot](https://cloud.baidu.com/doc/WENXINWORKSHOP/s/jlil56u11) 是由百度开发的大型语言模型，涵盖了大量中文数据。 本笔记本介绍如何开始使用 ErnieBot 聊天模型。

**弃用警告**

我们建议用户使用 `langchain_community.chat_models.ErnieBotChat` 的用户改用 `langchain_community.chat_models.QianfanChatEndpoint`。

`QianfanChatEndpoint` 的文档在 [这里](/docs/integrations/chat/baidu_qianfan_endpoint/)。

我们推荐用户使用 `QianfanChatEndpoint` 的原因有四个：

1. `QianfanChatEndpoint` 支持更多的 Qianfan 平台上的 LLM。
2. `QianfanChatEndpoint` 支持流式模式。
3. `QianfanChatEndpoint` 支持功能调用使用。
4. `ErnieBotChat` 缺乏维护且已弃用。

迁移的一些提示：

- 将 `ernie_client_id` 更改为 `qianfan_ak`，同时将 `ernie_client_secret` 更改为 `qianfan_sk`。
- 安装 `qianfan` 包，例如 `pip install qianfan`
- 将 `ErnieBotChat` 更改为 `QianfanChatEndpoint`。


```python
from langchain_community.chat_models.baidu_qianfan_endpoint import QianfanChatEndpoint

chat = QianfanChatEndpoint(
    qianfan_ak="your qianfan ak",
    qianfan_sk="your qianfan sk",
)
```

## 使用方法


```python
from langchain_community.chat_models import ErnieBotChat
from langchain_core.messages import HumanMessage

chat = ErnieBotChat(
    ernie_client_id="YOUR_CLIENT_ID", ernie_client_secret="YOUR_CLIENT_SECRET"
)
```

或者你可以在环境变量中设置 `client_id` 和 `client_secret`
```bash
export ERNIE_CLIENT_ID=YOUR_CLIENT_ID
export ERNIE_CLIENT_SECRET=YOUR_CLIENT_SECRET
```


```python
chat([HumanMessage(content="hello there, who are you?")])
```



```output
AIMessage(content='Hello, I am an artificial intelligence language model. My purpose is to help users answer questions or provide information. What can I do for you?', additional_kwargs={}, example=False)
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)