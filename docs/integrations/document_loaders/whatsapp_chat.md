---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/whatsapp_chat.ipynb
---

# WhatsApp聊天

>[WhatsApp](https://www.whatsapp.com/)（也称为`WhatsApp Messenger`）是一款免费、跨平台的集中式即时消息（IM）和语音通信（VoIP）服务。它允许用户发送文本和语音消息，进行语音和视频通话，并分享图片、文档、用户位置及其他内容。

本笔记本涵盖如何将`WhatsApp Chats`中的数据加载到可以被LangChain处理的格式中。


```python
from langchain_community.document_loaders import WhatsAppChatLoader
```


```python
loader = WhatsAppChatLoader("example_data/whatsapp_chat.txt")
```


```python
loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)