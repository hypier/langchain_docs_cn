---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/postgres_chat_message_history.ipynb
---

# Postgres

>[PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL)，也称为 `Postgres`，是一个免费的开源关系数据库管理系统（RDBMS），强调可扩展性和SQL合规性。

本笔记本介绍如何使用Postgres存储聊天消息历史记录。


```python
from langchain_community.chat_message_histories import (
    PostgresChatMessageHistory,
)

history = PostgresChatMessageHistory(
    connection_string="postgresql://postgres:mypassword@localhost/chat_history",
    session_id="foo",
)

history.add_user_message("hi!")

history.add_ai_message("whats up?")
```


```python
history.messages
```