---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat_loaders/telegram.ipynb
---

# Telegram

本笔记展示了如何使用 Telegram 聊天加载器。此类帮助将导出的 Telegram 对话映射到 LangChain 聊天消息。

该过程分为三个步骤：
1. 通过从 Telegram 应用中复制聊天记录并粘贴到本地计算机的文件中来导出聊天 .txt 文件
2. 创建指向 json 文件或 JSON 文件目录的文件路径的 `TelegramChatLoader`
3. 调用 `loader.load()`（或 `loader.lazy_load()`）来执行转换。可选地使用 `merge_chat_runs` 将同一发件人的消息按顺序合并，和/或使用 `map_ai_messages` 将指定发件人的消息转换为 "AIMessage" 类。

## 1. 创建消息转储

目前（2023/08/23），此加载器最佳支持从[Telegram桌面应用](https://desktop.telegram.org/)导出的聊天历史生成的json文件格式。

**重要：** 有一些“轻量版”的Telegram，如“MacOS版Telegram”，缺少导出功能。请确保使用正确的应用程序导出文件。

导出步骤：
1. 下载并打开Telegram桌面版
2. 选择一个对话
3. 进入对话设置（目前在右上角的三个点）
4. 点击“导出聊天历史”
5. 取消选择照片和其他媒体。选择“机器可读的JSON”格式进行导出。

下面是一个示例： 

```python
%%writefile telegram_conversation.json
{
 "name": "Jiminy",
 "type": "personal_chat",
 "id": 5965280513,
 "messages": [
  {
   "id": 1,
   "type": "message",
   "date": "2023-08-23T13:11:23",
   "date_unixtime": "1692821483",
   "from": "Jiminy Cricket",
   "from_id": "user123450513",
   "text": "You better trust your conscience",
   "text_entities": [
    {
     "type": "plain",
     "text": "You better trust your conscience"
    }
   ]
  },
  {
   "id": 2,
   "type": "message",
   "date": "2023-08-23T13:13:20",
   "date_unixtime": "1692821600",
   "from": "Batman & Robin",
   "from_id": "user6565661032",
   "text": "What did you just say?",
   "text_entities": [
    {
     "type": "plain",
     "text": "What did you just say?"
    }
   ]
  }
 ]
}
```
```output
Overwriting telegram_conversation.json
```

## 2. 创建聊天加载器

所需的仅是文件路径。您可以选择性地指定与 AI 消息对应的用户名，以及配置是否合并消息运行。

```python
from langchain_community.chat_loaders.telegram import TelegramChatLoader
```

```python
loader = TelegramChatLoader(
    path="./telegram_conversation.json",
)
```

## 3. 加载消息

`load()`（或 `lazy_load`）方法返回一个“ChatSessions”列表，该列表当前仅包含每个加载会话的消息列表。

```python
from typing import List

from langchain_community.chat_loaders.utils import (
    map_ai_messages,
    merge_chat_runs,
)
from langchain_core.chat_sessions import ChatSession

raw_messages = loader.lazy_load()
# Merge consecutive messages from the same sender into a single message
merged_messages = merge_chat_runs(raw_messages)
# Convert messages from "Jiminy Cricket" to AI messages
messages: List[ChatSession] = list(
    map_ai_messages(merged_messages, sender="Jiminy Cricket")
)
```

### 下一步

您可以根据需要使用这些消息，例如微调模型、选择少量示例，或直接进行下一条消息的预测。  

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI()

for chunk in llm.stream(messages[0]["messages"]):
    print(chunk.content, end="", flush=True)
```
```output
I said, "You better trust your conscience."
```