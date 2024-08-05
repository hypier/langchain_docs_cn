---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/momento_chat_message_history.ipynb
---

# Momento Cache

>[Momento Cache](https://docs.momentohq.com/) 是全球首个真正无服务器的缓存服务。它提供即时弹性、零扩展能力和超快性能。

本笔记本介绍如何使用 [Momento Cache](https://www.gomomento.com/services/cache) 来存储聊天消息历史，使用 `MomentoChatMessageHistory` 类。有关如何设置 Momento 的更多详细信息，请参阅 Momento [文档](https://docs.momentohq.com/getting-started)。

请注意，默认情况下，如果给定名称的缓存尚不存在，我们将创建一个缓存。

您需要获取一个 Momento API 密钥才能使用此类。您可以将其作为命名参数 `api_key` 传递给 `MomentoChatMessageHistory.from_client_params`，或者直接传递给 momento.CacheClient，或者将其设置为环境变量 `MOMENTO_API_KEY`。

```python
from datetime import timedelta

from langchain_community.chat_message_histories import MomentoChatMessageHistory

session_id = "foo"
cache_name = "langchain"
ttl = timedelta(days=1)
history = MomentoChatMessageHistory.from_client_params(
    session_id,
    cache_name,
    ttl,
)

history.add_user_message("hi!")

history.add_ai_message("whats up?")
```

```python
history.messages
```

```output
[HumanMessage(content='hi!', additional_kwargs={}, example=False),
 AIMessage(content='whats up?', additional_kwargs={}, example=False)]
```