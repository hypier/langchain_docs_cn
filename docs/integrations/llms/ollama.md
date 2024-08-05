---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/ollama.ipynb
sidebar_label: Ollama
---

# OllamaLLM

:::caution
您当前正在查看有关使用Ollama模型作为[文本补全模型](/docs/concepts/#llms)的文档。许多流行的Ollama模型是[聊天补全模型](/docs/concepts/#chat-models)。

您可能想查看[此页面](/docs/integrations/chat/ollama/)。
:::

本页面介绍了如何使用LangChain与`Ollama`模型进行交互。

## 安装


```python
# install package
%pip install -U langchain-ollama
```

## 设置

首先，按照 [这些说明](https://github.com/jmorganca/ollama) 设置并运行本地 Ollama 实例：

* [下载](https://ollama.ai/download) 并安装 Ollama 到可用的支持平台（包括 Windows 子系统 Linux）
* 通过 `ollama pull <name-of-model>` 获取可用的 LLM 模型
    * 通过 [模型库](https://ollama.ai/library) 查看可用模型的列表
    * 例如，`ollama pull llama3`
* 这将下载模型的默认标记版本。通常，默认指向最新的、参数最小的模型。

> 在 Mac 上，模型将下载到 `~/.ollama/models`
> 
> 在 Linux（或 WSL）上，模型将存储在 `/usr/share/ollama/.ollama/models`

* 指定感兴趣模型的确切版本，如 `ollama pull vicuna:13b-v1.5-16k-q4_0`（在此实例中查看 `Vicuna` 模型的 [各种标签](https://ollama.ai/library/vicuna/tags)）
* 要查看所有已拉取的模型，请使用 `ollama list`
* 要直接从命令行与模型聊天，请使用 `ollama run <name-of-model>`
* 查看 [Ollama 文档](https://github.com/jmorganca/ollama) 以获取更多命令。在终端中运行 `ollama help` 也可以查看可用命令。

## 用法


```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_ollama.llms import OllamaLLM

template = """Question: {question}

Answer: Let's think step by step."""

prompt = ChatPromptTemplate.from_template(template)

model = OllamaLLM(model="llama3")

chain = prompt | model

chain.invoke({"question": "What is LangChain?"})
```



```output
'A great start!\n\nLangChain is a type of AI model that uses language processing techniques to generate human-like text based on input prompts or chains of reasoning. In other words, it can have a conversation with humans, understanding the context and responding accordingly.\n\nHere\'s a possible breakdown:\n\n* "Lang" likely refers to its focus on natural language processing (NLP) and linguistic analysis.\n* "Chain" suggests that LangChain is designed to generate text in response to a series of connected ideas or prompts, rather than simply generating random text.\n\nSo, what do you think LangChain\'s capabilities might be?'
```

## 多模态

Ollama 支持多模态 LLM，例如 [bakllava](https://ollama.com/library/bakllava) 和 [llava](https://ollama.com/library/llava)。

    ollama pull bakllava

确保更新 Ollama，以便您拥有最新版本以支持多模态。


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
    Display base64 encoded string as image

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


```python
from langchain_ollama import OllamaLLM

llm = OllamaLLM(model="bakllava")

llm_with_image_context = llm.bind(images=[image_b64])
llm_with_image_context.invoke("What is the dollar based gross retention rate:")
```



```output
'90%'
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)