---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat_loaders/langsmith_dataset.ipynb
---

# LangSmith 聊天数据集

本笔记本演示了一种简单的方法来加载 LangSmith 聊天数据集并在该数据上微调模型。这个过程简单，包含 3 个步骤。

1. 创建聊天数据集。
2. 使用 LangSmithDatasetChatLoader 加载示例。
3. 微调你的模型。

然后你可以在你的 LangChain 应用中使用微调后的模型。

在开始之前，让我们安装必要的依赖。

## 前提条件

确保您已安装 langchain >= 0.0.311，并已使用您的 LangSmith API 密钥配置了环境。


```python
%pip install --upgrade --quiet  langchain langchain-openai
```


```python
import os
import uuid

uid = uuid.uuid4().hex[:6]
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "YOUR API KEY"
```

## 1. 选择数据集

本笔记本直接在选择要微调的运行上微调模型。您通常会从跟踪的运行中整理这些。您可以在文档中了解更多有关 LangSmith 数据集的信息 [docs](https://docs.smith.langchain.com/evaluation/concepts#datasets)。

为了本教程的目的，我们将在这里上传一个现有的数据集供您使用。


```python
from langsmith.client import Client

client = Client()
```


```python
import requests

url = "https://raw.githubusercontent.com/langchain-ai/langchain/master/docs/docs/integrations/chat_loaders/example_data/langsmith_chat_dataset.json"
response = requests.get(url)
response.raise_for_status()
data = response.json()
```


```python
dataset_name = f"Extraction Fine-tuning Dataset {uid}"
ds = client.create_dataset(dataset_name=dataset_name, data_type="chat")
```


```python
_ = client.create_examples(
    inputs=[e["inputs"] for e in data],
    outputs=[e["outputs"] for e in data],
    dataset_id=ds.id,
)
```

## 2. 准备数据
现在我们可以创建一个 LangSmithRunChatLoader 的实例，并使用其 lazy_load() 方法加载聊天会话。

```python
from langchain_community.chat_loaders.langsmith import LangSmithDatasetChatLoader

loader = LangSmithDatasetChatLoader(dataset_name=dataset_name)

chat_sessions = loader.lazy_load()
```

#### 加载聊天会话后，将其转换为适合微调的格式。

```python
from langchain_community.adapters.openai import convert_messages_for_finetuning

training_data = convert_messages_for_finetuning(chat_sessions)
```

## 3. 微调模型
现在，使用 OpenAI 库启动微调过程。

```python
import json
import time
from io import BytesIO

import openai

my_file = BytesIO()
for dialog in training_data:
    my_file.write((json.dumps({"messages": dialog}) + "\n").encode("utf-8"))

my_file.seek(0)
training_file = openai.files.create(file=my_file, purpose="fine-tune")

job = openai.fine_tuning.jobs.create(
    training_file=training_file.id,
    model="gpt-3.5-turbo",
)

# 等待微调完成（这可能需要一些时间）
status = openai.fine_tuning.jobs.retrieve(job.id).status
start_time = time.time()
while status != "succeeded":
    print(f"Status=[{status}]... {time.time() - start_time:.2f}s", end="\r", flush=True)
    time.sleep(5)
    status = openai.fine_tuning.jobs.retrieve(job.id).status

# 现在你的模型已完成微调！
```
```output
Status=[running]... 429.55s. 46.34s
```

## 4. 在 LangChain 中的使用

在微调后，使用生成的模型 ID 与您的 LangChain 应用中的 ChatOpenAI 模型类。

```python
# Get the fine-tuned model ID
job = openai.fine_tuning.jobs.retrieve(job.id)
model_id = job.fine_tuned_model

# Use the fine-tuned model in LangChain
from langchain_openai import ChatOpenAI

model = ChatOpenAI(
    model=model_id,
    temperature=1,
)
```

```python
model.invoke("There were three ravens sat on a tree.")
```

```output
AIMessage(content='[{"s": "There were three ravens", "object": "tree", "relation": "sat on"}, {"s": "three ravens", "object": "a tree", "relation": "sat on"}]')
```

现在您已经成功使用来自 LangSmith LLM 运行的数据微调了一个模型！