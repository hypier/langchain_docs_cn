---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/streamlit_chat_message_history.ipynb
---

# Streamlit

>[Streamlit](https://docs.streamlit.io/) 是一个开源的 Python 库，使创建和共享美观的自定义 Web 应用程序变得简单，适用于机器学习和数据科学。

本笔记本介绍了如何在 `Streamlit` 应用中存储和使用聊天消息历史。`StreamlitChatMessageHistory` 将在指定的 `key=` 中存储消息到
[Streamlit 会话状态](https://docs.streamlit.io/library/api-reference/session-state)中。默认键为 `"langchain_messages"`。

- 注意，`StreamlitChatMessageHistory` 仅在 Streamlit 应用中运行时有效。
- 您可能还对 LangChain 的 [StreamlitCallbackHandler](/docs/integrations/callbacks/streamlit) 感兴趣。
- 有关 Streamlit 的更多信息，请查看他们的
[入门文档](https://docs.streamlit.io/library/get-started)。

该集成位于 `langchain-community` 包中，因此我们需要安装它。我们还需要安装 `streamlit`。

```
pip install -U langchain-community streamlit
```

您可以在这里查看 [完整的应用示例](https://langchain-st-memory.streamlit.app/)，以及更多示例在
[github.com/langchain-ai/streamlit-agent](https://github.com/langchain-ai/streamlit-agent)。

```python
from langchain_community.chat_message_histories import (
    StreamlitChatMessageHistory,
)

history = StreamlitChatMessageHistory(key="chat_messages")

history.add_user_message("hi!")
history.add_ai_message("whats up?")
```

```python
history.messages
```

我们可以轻松地将此消息历史类与 [LCEL Runnables](/docs/how_to/message_history) 结合使用。

历史记录将在给定用户会话内的 Streamlit 应用的重新运行中持续存在。给定的 `StreamlitChatMessageHistory` 不会在用户会话之间持久化或共享。

```python
# 可选，指定您自己的 session_state 键以存储消息
msgs = StreamlitChatMessageHistory(key="special_app_key")

if len(msgs.messages) == 0:
    msgs.add_ai_message("How can I help you?")
```

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are an AI chatbot having a conversation with a human."),
        MessagesPlaceholder(variable_name="history"),
        ("human", "{question}"),
    ]
)

chain = prompt | ChatOpenAI()
```

```python
chain_with_history = RunnableWithMessageHistory(
    chain,
    lambda session_id: msgs,  # 始终返回之前创建的实例
    input_messages_key="question",
    history_messages_key="history",
)
```

对话式 Streamlit 应用通常会在每次重新运行时重新绘制每个先前的聊天消息。这可以通过迭代 `StreamlitChatMessageHistory.messages` 来轻松实现：

```python
import streamlit as st

for msg in msgs.messages:
    st.chat_message(msg.type).write(msg.content)

if prompt := st.chat_input():
    st.chat_message("human").write(prompt)

    # 和往常一样，当调用链时，新消息会添加到 StreamlitChatMessageHistory 中。
    config = {"configurable": {"session_id": "any"}}
    response = chain_with_history.invoke({"question": prompt}, config)
    st.chat_message("ai").write(response.content)
```

**[查看最终应用](https://langchain-st-memory.streamlit.app/)。**