---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/singlestoredb_chat_message_history.ipynb
---

# SingleStoreDB

本笔记本介绍如何使用 SingleStoreDB 存储聊天消息历史记录。


```python
from langchain_community.chat_message_histories import (
    SingleStoreDBChatMessageHistory,
)

history = SingleStoreDBChatMessageHistory(
    session_id="foo", host="root:pass@localhost:3306/db"
)

history.add_user_message("hi!")

history.add_ai_message("whats up?")
```


```python
history.messages
```