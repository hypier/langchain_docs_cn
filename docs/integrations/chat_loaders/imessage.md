---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat_loaders/imessage.ipynb
---

# iMessage

本笔记本展示了如何使用 iMessage 聊天加载器。该类帮助将 iMessage 对话转换为 LangChain 聊天消息。

在 MacOS 上，iMessage 将对话存储在 `~/Library/Messages/chat.db` 的 sqlite 数据库中（至少适用于 macOS Ventura 13.4）。 
`IMessageChatLoader` 从此数据库文件中加载数据。

1. 创建 `IMessageChatLoader`，并将文件路径指向您想要处理的 `chat.db` 数据库。
2. 调用 `loader.load()`（或 `loader.lazy_load()`）以执行转换。可选地使用 `merge_chat_runs` 来按顺序合并来自同一发送方的消息，和/或使用 `map_ai_messages` 将指定发送方的消息转换为 "AIMessage" 类。

## 1. 访问聊天数据库

您的终端可能无法访问 `~/Library/Messages`。要使用此类，您可以将数据库复制到可访问的目录（例如，文档）并从那里加载。或者（不推荐），您可以在系统设置 > 安全性与隐私 > 完整磁盘访问中为您的终端模拟器授予完全磁盘访问权限。

我们创建了一个示例数据库，您可以在 [此链接的驱动器文件](https://drive.google.com/file/d/1NebNKqTA2NXApCmeH6mu0unJD2tANZzo/view?usp=sharing) 中使用。

```python
# This uses some example data
import requests


def download_drive_file(url: str, output_path: str = "chat.db") -> None:
    file_id = url.split("/")[-2]
    download_url = f"https://drive.google.com/uc?export=download&id={file_id}"

    response = requests.get(download_url)
    if response.status_code != 200:
        print("Failed to download the file.")
        return

    with open(output_path, "wb") as file:
        file.write(response.content)
        print(f"File {output_path} downloaded.")


url = (
    "https://drive.google.com/file/d/1NebNKqTA2NXApCmeH6mu0unJD2tANZzo/view?usp=sharing"
)

# Download file to chat.db
download_drive_file(url)
```
```output
File chat.db downloaded.
```

## 2. 创建聊天加载器

提供加载器与 zip 目录的文件路径。您可以选择性地指定映射到 AI 消息的用户 ID，以及配置是否合并消息运行。

```python
from langchain_community.chat_loaders.imessage import IMessageChatLoader
```

```python
loader = IMessageChatLoader(
    path="./chat.db",
)
```

## 3. 加载消息

`load()`（或 `lazy_load`）方法返回一个“ChatSessions”列表，该列表当前仅包含每个加载的对话的消息列表。所有消息最初都映射到“HumanMessage”对象。

您可以选择合并消息“运行”（来自同一发送者的连续消息），并选择一个发送者来代表“AI”。经过微调的 LLM 将学习生成这些 AI 消息。

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
# Convert messages from "Tortoise" to AI messages. Do you have a guess who these conversations are between?
chat_sessions: List[ChatSession] = list(
    map_ai_messages(merged_messages, sender="Tortoise")
)
```

```python
# Now all of the Tortoise's messages will take the AI message class
# which maps to the 'assistant' role in OpenAI's training format
chat_sessions[0]["messages"][:3]
```

```output
[AIMessage(content="Slow and steady, that's my motto.", additional_kwargs={'message_time': 1693182723, 'sender': 'Tortoise'}, example=False),
 HumanMessage(content='Speed is key!', additional_kwargs={'message_time': 1693182753, 'sender': 'Hare'}, example=False),
 AIMessage(content='A balanced approach is more reliable.', additional_kwargs={'message_time': 1693182783, 'sender': 'Tortoise'}, example=False)]
```

## 3. 准备微调

现在是将我们的聊天消息转换为 OpenAI 字典的时刻。我们可以使用 `convert_messages_for_finetuning` 工具来完成此操作。

```python
from langchain_community.adapters.openai import convert_messages_for_finetuning
```

```python
training_data = convert_messages_for_finetuning(chat_sessions)
print(f"Prepared {len(training_data)} dialogues for training")
```
```output
Prepared 10 dialogues for training
```

## 4. 微调模型

现在是微调模型的时候了。确保您已安装 `openai` 并已正确设置 `OPENAI_API_KEY`


```python
%pip install --upgrade --quiet  langchain-openai
```


```python
import json
import time
from io import BytesIO

import openai

# We will write the jsonl file in memory
my_file = BytesIO()
for m in training_data:
    my_file.write((json.dumps({"messages": m}) + "\n").encode("utf-8"))

my_file.seek(0)
training_file = openai.files.create(file=my_file, purpose="fine-tune")

# OpenAI audits each training file for compliance reasons.
# This make take a few minutes
status = openai.files.retrieve(training_file.id).status
start_time = time.time()
while status != "processed":
    print(f"Status=[{status}]... {time.time() - start_time:.2f}s", end="\r", flush=True)
    time.sleep(5)
    status = openai.files.retrieve(training_file.id).status
print(f"File {training_file.id} ready after {time.time() - start_time:.2f} seconds.")
```
```output
File file-zHIgf4r8LltZG3RFpkGd4Sjf ready after 10.19 seconds.
```
文件准备好后，接下来是启动训练任务。


```python
job = openai.fine_tuning.jobs.create(
    training_file=training_file.id,
    model="gpt-3.5-turbo",
)
```

在您的模型准备期间，喝杯茶吧。这可能需要一些时间！


```python
status = openai.fine_tuning.jobs.retrieve(job.id).status
start_time = time.time()
while status != "succeeded":
    print(f"Status=[{status}]... {time.time() - start_time:.2f}s", end="\r", flush=True)
    time.sleep(5)
    job = openai.fine_tuning.jobs.retrieve(job.id)
    status = job.status
```
```output
Status=[running]... 524.95s
```

```python
print(job.fine_tuned_model)
```
```output
ft:gpt-3.5-turbo-0613:personal::7sKoRdlz
```

## 5. 在 LangChain 中使用

您可以直接在 `ChatOpenAI` 模型类中使用生成的模型 ID。

```python
from langchain_openai import ChatOpenAI

model = ChatOpenAI(
    model=job.fine_tuned_model,
    temperature=1,
)
```

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "You are speaking to hare."),
        ("human", "{input}"),
    ]
)

chain = prompt | model | StrOutputParser()
```

```python
for tok in chain.stream({"input": "What's the golden thread?"}):
    print(tok, end="", flush=True)
```
```output
A symbol of interconnectedness.
```