---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/telegram.ipynb
---

# Telegram

>[Telegram Messenger](https://web.telegram.org/a/) 是一个全球可访问的增值服务、跨平台、加密、基于云的集中式即时消息服务。该应用程序还提供可选的端到端加密聊天、视频通话、VoIP、文件共享及其他多个功能。

本笔记本涵盖如何将数据从 `Telegram` 加载为可以被 LangChain 吞噬的格式。

```python
from langchain_community.document_loaders import (
    TelegramChatApiLoader,
    TelegramChatFileLoader,
)
```

```python
loader = TelegramChatFileLoader("example_data/telegram.json")
```

```python
loader.load()
```

```output
[Document(page_content="Henry on 2020-01-01T00:00:02: It's 2020...\n\nHenry on 2020-01-01T00:00:04: Fireworks!\n\nGrace ðŸ§¤ ðŸ\x8d’ on 2020-01-01T00:00:05: You're a minute late!\n\n", metadata={'source': 'example_data/telegram.json'})]
```

`TelegramChatApiLoader` 直接从 Telegram 的任何指定聊天中加载数据。要导出数据，您需要验证您的 Telegram 账户。

您可以从 https://my.telegram.org/auth?to=apps 获取 API_HASH 和 API_ID。

chat_entity – 推荐使用频道的 [entity](https://docs.telethon.dev/en/stable/concepts/entities.html?highlight=Entity#what-is-an-entity)。

```python
loader = TelegramChatApiLoader(
    chat_entity="<CHAT_URL>",  # 推荐在此使用 Entity
    api_hash="<API HASH >",
    api_id="<API_ID>",
    username="",  # 仅在缓存会话时需要。
)
```

```python
loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)