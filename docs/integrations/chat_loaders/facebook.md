---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat_loaders/facebook.ipynb
---

# Facebook Messenger

此笔记本展示了如何以可微调的格式从 Facebook 加载数据。总体步骤如下：

1. 将您的 Messenger 数据下载到磁盘。
2. 创建聊天加载器并调用 `loader.load()`（或 `loader.lazy_load()`）以执行转换。
3. 可选地使用 `merge_chat_runs` 将来自同一发送者的消息按顺序合并，和/或使用 `map_ai_messages` 将指定发送者的消息转换为 "AIMessage" 类。完成后，调用 `convert_messages_for_finetuning` 准备您的数据以进行微调。

完成上述步骤后，您可以微调您的模型。为此，您需要完成以下步骤：

4. 将您的消息上传到 OpenAI 并运行微调作业。
6. 在您的 LangChain 应用中使用生成的模型！

让我们开始吧。

## 1. 下载数据

要下载自己的消息数据，请按照[这里](https://www.zapptales.com/en/download-facebook-messenger-chat-history-how-to/)的说明进行操作。重要提示 - 确保以 JSON 格式下载（而不是 HTML）。

我们在[这个谷歌云盘链接](https://drive.google.com/file/d/1rh1s1o2i7B-Sk1v9o8KNgivLVGwJ-osV/view?usp=sharing)上托管了一个示例数据包，我们将在本指南中使用它。


```python
# This uses some example data
import zipfile

import requests


def download_and_unzip(url: str, output_path: str = "file.zip") -> None:
    file_id = url.split("/")[-2]
    download_url = f"https://drive.google.com/uc?export=download&id={file_id}"

    response = requests.get(download_url)
    if response.status_code != 200:
        print("Failed to download the file.")
        return

    with open(output_path, "wb") as file:
        file.write(response.content)
        print(f"File {output_path} downloaded.")

    with zipfile.ZipFile(output_path, "r") as zip_ref:
        zip_ref.extractall()
        print(f"File {output_path} has been unzipped.")


# URL of the file to download
url = (
    "https://drive.google.com/file/d/1rh1s1o2i7B-Sk1v9o8KNgivLVGwJ-osV/view?usp=sharing"
)

# Download and unzip
download_and_unzip(url)
```
```output
File file.zip downloaded.
File file.zip has been unzipped.
```

## 2. 创建聊天加载器

我们有两个不同的 `FacebookMessengerChatLoader` 类，一个用于整个聊天目录，另一个用于加载单个文件。我们

```python
directory_path = "./hogwarts"
```

```python
from langchain_community.chat_loaders.facebook_messenger import (
    FolderFacebookMessengerChatLoader,
    SingleFileFacebookMessengerChatLoader,
)
```

```python
loader = SingleFileFacebookMessengerChatLoader(
    path="./hogwarts/inbox/HermioneGranger/messages_Hermione_Granger.json",
)
```

```python
chat_session = loader.load()[0]
chat_session["messages"][:3]
```

```output
[HumanMessage(content="Hi Hermione! How's your summer going so far?", additional_kwargs={'sender': 'Harry Potter'}),
 HumanMessage(content="Harry! Lovely to hear from you. My summer is going well, though I do miss everyone. I'm spending most of my time going through my books and researching fascinating new topics. How about you?", additional_kwargs={'sender': 'Hermione Granger'}),
 HumanMessage(content="I miss you all too. The Dursleys are being their usual unpleasant selves but I'm getting by. At least I can practice some spells in my room without them knowing. Let me know if you find anything good in your researching!", additional_kwargs={'sender': 'Harry Potter'})]
```

```python
loader = FolderFacebookMessengerChatLoader(
    path="./hogwarts",
)
```

```python
chat_sessions = loader.load()
len(chat_sessions)
```

```output
9
```

## 3. 准备微调

调用 `load()` 返回我们可以提取的所有聊天消息作为人类消息。在与聊天机器人对话时，谈话通常遵循相对真实对话更严格的交替对话模式。

您可以选择合并消息“运行”（来自同一发送者的连续消息），并选择一个发送者来代表“AI”。微调后的 LLM 将学习生成这些 AI 消息。

```python
from langchain_community.chat_loaders.utils import (
    map_ai_messages,
    merge_chat_runs,
)
```

```python
merged_sessions = merge_chat_runs(chat_sessions)
alternating_sessions = list(map_ai_messages(merged_sessions, "Harry Potter"))
```

```python
# 现在所有哈利·波特的消息将采用 AI 消息类
# 该类映射到 OpenAI 训练格式中的 'assistant' 角色
alternating_sessions[0]["messages"][:3]
```

```output
[AIMessage(content="Professor Snape, I was hoping I could speak with you for a moment about something that's been concerning me lately.", additional_kwargs={'sender': 'Harry Potter'}),
 HumanMessage(content="What is it, Potter? I'm quite busy at the moment.", additional_kwargs={'sender': 'Severus Snape'}),
 AIMessage(content="I apologize for the interruption, sir. I'll be brief. I've noticed some strange activity around the school grounds at night. I saw a cloaked figure lurking near the Forbidden Forest last night. I'm worried someone may be plotting something sinister.", additional_kwargs={'sender': 'Harry Potter'})]
```

#### 现在我们可以转换为 OpenAI 格式字典

```python
from langchain_community.adapters.openai import convert_messages_for_finetuning
```

```python
training_data = convert_messages_for_finetuning(alternating_sessions)
print(f"Prepared {len(training_data)} dialogues for training")
```
```output
Prepared 9 dialogues for training
```

```python
training_data[0][:3]
```

```output
[{'role': 'assistant',
  'content': "Professor Snape, I was hoping I could speak with you for a moment about something that's been concerning me lately."},
 {'role': 'user',
  'content': "What is it, Potter? I'm quite busy at the moment."},
 {'role': 'assistant',
  'content': "I apologize for the interruption, sir. I'll be brief. I've noticed some strange activity around the school grounds at night. I saw a cloaked figure lurking near the Forbidden Forest last night. I'm worried someone may be plotting something sinister."}]
```

OpenAI 当前要求微调作业至少有 10 个训练示例，尽管他们建议大多数任务在 50-100 之间。由于我们只有 9 个聊天会话，我们可以将它们细分（可选地有一些重叠），使每个训练示例由整个对话的一部分组成。

Facebook 聊天会话（每人 1 个）通常跨越多天和多次对话，因此长距离依赖性可能并不那么重要。

```python
# 我们的聊天是交替的，我们将每个数据点设置为 8 条消息的组，
# 其中 2 条消息重叠
chunk_size = 8
overlap = 2

training_examples = [
    conversation_messages[i : i + chunk_size]
    for conversation_messages in training_data
    for i in range(0, len(conversation_messages) - chunk_size + 1, chunk_size - overlap)
]

len(training_examples)
```

```output
100
```

## 4. 微调模型

现在是微调模型的时候了。确保你已经安装了 `openai` 并且正确设置了 `OPENAI_API_KEY`


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
for m in training_examples:
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
File file-ULumAXLEFw3vB6bb9uy6DNVC ready after 0.00 seconds.
```
文件准备好后，是时候启动训练任务了。


```python
job = openai.fine_tuning.jobs.create(
    training_file=training_file.id,
    model="gpt-3.5-turbo",
)
```

在模型准备的过程中，喝杯茶吧。这可能需要一些时间！


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
Status=[running]... 874.29s. 56.93s
```

```python
print(job.fine_tuned_model)
```
```output
ft:gpt-3.5-turbo-0613:personal::8QnAzWMr
```

## 5. 在 LangChain 中的使用

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
        ("human", "{input}"),
    ]
)

chain = prompt | model | StrOutputParser()
```

```python
for tok in chain.stream({"input": "What classes are you taking?"}):
    print(tok, end="", flush=True)
```
```output
I'm taking Charms, Defense Against the Dark Arts, Herbology, Potions, Transfiguration, and Ancient Runes. How about you?
```