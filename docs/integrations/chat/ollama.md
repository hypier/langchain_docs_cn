---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/ollama.ipynb
sidebar_label: Ollama
---

# ChatOllama

[Ollama](https://ollama.ai/) 允许您在本地运行开源大型语言模型，例如 Llama 2。

Ollama 将模型权重、配置和数据打包成一个由 Modelfile 定义的单一包。

它优化了设置和配置细节，包括 GPU 使用。

有关支持的模型及模型变体的完整列表，请参见 [Ollama 模型库](https://github.com/jmorganca/ollama#model-library)。

## 概述

### 集成细节

| 类 | 包 | 本地 | 可序列化 | [JS 支持](https://js.langchain.com/v0.2/docs/integrations/chat/ollama) | 包下载量 | 包最新版本 |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [ChatOllama](https://api.python.langchain.com/en/latest/chat_models/langchain_ollama.chat_models.ChatOllama.html) | [langchain-ollama](https://api.python.langchain.com/en/latest/ollama_api_reference.html) | ✅ | ❌ | ✅ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-ollama?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain-ollama?style=flat-square&label=%20) |

### 模型特性
| [工具调用](/docs/how_to/tool_calling/) | [结构化输出](/docs/how_to/structured_output/) | JSON 模式 | [图像输入](/docs/how_to/multimodal_inputs/) | 音频输入 | 视频输入 | [令牌级流式传输](/docs/how_to/chat_streaming/) | 原生异步 | [令牌使用](/docs/how_to/chat_token_usage_tracking/) | [日志概率](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ | ✅ | ❌ | ❌ |

## 设置

首先，请按照 [这些说明](https://github.com/jmorganca/ollama) 设置并运行本地的 Ollama 实例：

* [下载](https://ollama.ai/download) 并安装 Ollama 到可用的支持平台（包括 Windows 子系统 Linux）
* 通过 `ollama pull <name-of-model>` 获取可用的 LLM 模型
    * 通过 [模型库](https://ollama.ai/library) 查看可用模型的列表
    * 例如，`ollama pull llama3`
* 这将下载该模型的默认标记版本。通常，默认指向最新、参数最小的模型。

> 在 Mac 上，模型将下载到 `~/.ollama/models`
> 
> 在 Linux（或 WSL）上，模型将存储在 `/usr/share/ollama/.ollama/models`

* 指定感兴趣模型的确切版本，如 `ollama pull vicuna:13b-v1.5-16k-q4_0`（在此实例中查看 `Vicuna` 模型的 [各种标签](https://ollama.ai/library/vicuna/tags)）
* 要查看所有已拉取的模型，请使用 `ollama list`
* 要直接从命令行与模型聊天，请使用 `ollama run <name-of-model>`
* 查看 [Ollama 文档](https://github.com/jmorganca/ollama) 获取更多命令。在终端中运行 `ollama help` 以查看可用命令。

如果您希望自动跟踪模型调用，您还可以通过取消注释下面的内容来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：

```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### 安装

LangChain Ollama 集成位于 `langchain-ollama` 包中：

```python
%pip install -qU langchain-ollama
```

## 实例化

现在我们可以实例化我们的模型对象并生成聊天补全：

- TODO: 使用相关参数更新模型实例化。


```python
from langchain_ollama import ChatOllama

llm = ChatOllama(
    model="llama3",
    temperature=0,
    # other params...
)
```

## 调用


```python
from langchain_core.messages import AIMessage

messages = [
    (
        "system",
        "You are a helpful assistant that translates English to French. Translate the user sentence.",
    ),
    ("human", "I love programming."),
]
ai_msg = llm.invoke(messages)
ai_msg
```



```output
AIMessage(content='Je adore le programmation.\n\n(Note: "programmation" is not commonly used in French, but I translated it as "le programmation" to maintain the same grammatical structure and meaning as the original English sentence.)', response_metadata={'model': 'llama3', 'created_at': '2024-07-22T17:43:54.731273Z', 'message': {'role': 'assistant', 'content': ''}, 'done_reason': 'stop', 'done': True, 'total_duration': 11094839375, 'load_duration': 10121854667, 'prompt_eval_count': 36, 'prompt_eval_duration': 146569000, 'eval_count': 46, 'eval_duration': 816593000}, id='run-befccbdc-e1f9-42a9-85cf-e69b926d6b8b-0', usage_metadata={'input_tokens': 36, 'output_tokens': 46, 'total_tokens': 82})
```



```python
print(ai_msg.content)
```
```output
Je adore le programmation.

(Note: "programmation" is not commonly used in French, but I translated it as "le programmation" to maintain the same grammatical structure and meaning as the original English sentence.)
```

## 链接

我们可以通过一个提示模板来[链式](/docs/how_to/sequence/)我们的模型，如下所示：

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant that translates {input_language} to {output_language}.",
        ),
        ("human", "{input}"),
    ]
)

chain = prompt | llm
chain.invoke(
    {
        "input_language": "English",
        "output_language": "German",
        "input": "I love programming.",
    }
)
```



```output
AIMessage(content='Ich liebe Programmieren!\n\n(Note: "Ich liebe" means "I love", "Programmieren" is the verb for "programming")', response_metadata={'model': 'llama3', 'created_at': '2024-07-04T04:22:33.864132Z', 'message': {'role': 'assistant', 'content': ''}, 'done_reason': 'stop', 'done': True, 'total_duration': 1310800083, 'load_duration': 1782000, 'prompt_eval_count': 16, 'prompt_eval_duration': 250199000, 'eval_count': 29, 'eval_duration': 1057192000}, id='run-cbadbe59-2de2-4ec0-a18a-b3220226c3d2-0')
```

## 工具调用

我们可以使用 [工具调用](https://blog.langchain.dev/improving-core-tool-interfaces-and-docs-in-langchain/) 与一个 [经过微调以用于工具使用的 LLM](https://ollama.com/library/llama3-groq-tool-use)： 

```
ollama pull llama3-groq-tool-use
```

我们可以直接将普通的 Python 函数作为工具传递。


```python
from typing import List

from langchain_ollama import ChatOllama
from typing_extensions import TypedDict


def validate_user(user_id: int, addresses: List) -> bool:
    """使用历史地址验证用户。

    参数：
        user_id: (int) 用户 ID。
        addresses: 之前的地址。
    """
    return True


llm = ChatOllama(
    model="llama3-groq-tool-use",
    temperature=0,
).bind_tools([validate_user])

result = llm.invoke(
    "你能验证用户 123 吗？他们以前住在 "
    "123 Fake St, Boston MA 和 234 Pretend Boulevard, "
    "Houston TX。"
)
result.tool_calls
```



```output
[{'name': 'validate_user',
  'args': {'addresses': ['123 Fake St, Boston MA',
    '234 Pretend Boulevard, Houston TX'],
   'user_id': 123},
  'id': 'fe2148d3-95fb-48e9-845a-4bfecc1f1f96',
  'type': 'tool_call'}]
```


我们查看 LangSmith 跟踪以确认工具调用已执行： 

https://smith.langchain.com/public/4169348a-d6be-45df-a7cf-032f6baa4697/r

特别是，跟踪显示了工具模式是如何填充的。

## 多模态

Ollama 支持多模态 LLM，例如 [bakllava](https://ollama.com/library/bakllava) 和 [llava](https://ollama.com/library/llava)。

    ollama pull bakllava

请确保更新 Ollama，以便您拥有支持多模态的最新版本。


```python
import base64
from io import BytesIO

from IPython.display import HTML, display
from PIL import Image


def convert_to_base64(pil_image):
    """
    Convert PIL images to Base64 encoded strings

    :param pil_image: PIL image
    :return: Re-sized Base64 string
    """

    buffered = BytesIO()
    pil_image.save(buffered, format="JPEG")  # You can change the format if needed
    img_str = base64.b64encode(buffered.getvalue()).decode("utf-8")
    return img_str


def plt_img_base64(img_base64):
    """
    Disply base64 encoded string as image

    :param img_base64:  Base64 string
    """
    # Create an HTML img tag with the base64 string as the source
    image_html = f'<img src="data:image/jpeg;base64,{img_base64}" />'
    # Display the image by rendering the HTML
    display(HTML(image_html))


file_path = "../../../static/img/ollama_example_img.jpg"
pil_image = Image.open(file_path)

image_b64 = convert_to_base64(pil_image)
plt_img_base64(image_b64)
```

```html
```


```python
from langchain_core.messages import HumanMessage
from langchain_ollama import ChatOllama

llm = ChatOllama(model="bakllava", temperature=0)


def prompt_func(data):
    text = data["text"]
    image = data["image"]

    image_part = {
        "type": "image_url",
        "image_url": f"data:image/jpeg;base64,{image}",
    }

    content_parts = []

    text_part = {"type": "text", "text": text}

    content_parts.append(image_part)
    content_parts.append(text_part)

    return [HumanMessage(content=content_parts)]


from langchain_core.output_parsers import StrOutputParser

chain = prompt_func | llm | StrOutputParser()

query_chain = chain.invoke(
    {"text": "What is the Dollar-based gross retention rate?", "image": image_b64}
)

print(query_chain)
```
```output
90%
```

## API 参考

有关所有 ChatOllama 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/chat_models/langchain_ollama.chat_models.ChatOllama.html

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)