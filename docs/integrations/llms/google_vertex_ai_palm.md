---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/google_vertex_ai_palm.ipynb
keywords: [gemini, vertex, VertexAI, gemini-pro]
---

# Google Cloud Vertex AI

:::caution
您当前正在查看有关 Google Vertex [文本补全模型](/docs/concepts/#llms) 的文档。许多 Google 模型是 [聊天补全模型](/docs/concepts/#chat-models)。

您可能想查看 [此页面](/docs/integrations/chat/google_vertex_ai_palm/)。
:::

**注意：** 这与 `Google 生成式 AI` 集成是分开的，它在 `Google Cloud` 上公开了 [Vertex AI 生成 API](https://cloud.google.com/vertex-ai/docs/generative-ai/learn/overview)。

VertexAI 提供了 Google Cloud 中所有可用的基础模型：
- Gemini for Text ( `gemini-1.0-pro` )
- Gemini with Multimodality ( `gemini-1.5-pro-001` 和 `gemini-pro-vision` )
- Palm 2 for Text ( `text-bison` )
- Codey for Code Generation ( `code-bison` )

要查看可用模型的完整和最新列表，请访问 [VertexAI 文档](https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/overview)

## 设置

根据默认设置，Google Cloud [不使用](https://cloud.google.com/vertex-ai/docs/generative-ai/data-governance#foundation_model_development) 客户数据来训练其基础模型，这是 Google Cloud 在 AI/ML 隐私承诺中的一部分。有关 Google 如何处理数据的更多详细信息，请参阅 [Google 的客户数据处理附录 (CDPA)](https://cloud.google.com/terms/data-processing-addendum)。

要使用 `Vertex AI Generative AI`，您必须安装 `langchain-google-vertexai` Python 包，并且需要：
- 为您的环境配置凭据（gcloud、工作负载身份等...）
- 将服务账户 JSON 文件的路径存储为 GOOGLE_APPLICATION_CREDENTIALS 环境变量

该代码库使用 `google.auth` 库，首先查找上述提到的应用凭据变量，然后查找系统级别的身份验证。

有关更多信息，请参见：
- https://cloud.google.com/docs/authentication/application-default-credentials#GAC
- https://googleapis.dev/python/google-auth/latest/reference/google.auth.html#module-google.auth


```python
%pip install --upgrade --quiet  langchain-core langchain-google-vertexai
```
```output
Note: you may need to restart the kernel to use updated packages.
```

## 用法

VertexAI 支持所有 [LLM](/docs/how_to#llms) 功能。


```python
from langchain_google_vertexai import VertexAI

# 使用模型
model = VertexAI(model_name="gemini-pro")
```

注意：您还可以指定 [Gemini 版本](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/model-versioning#gemini-model-versions)


```python
# 指定特定模型版本
model = VertexAI(model_name="gemini-1.0-pro-002")
```


```python
message = "Python 作为编程语言有哪些优缺点？"
model.invoke(message)
```



```output
```



```python
await model.ainvoke(message)
```



```output
```



```python
for chunk in model.stream(message):
    print(chunk, end="", flush=True)
```
```output
## Python 的优缺点

### 优点：

* **易于学习和阅读**：Python 的语法清晰简洁，比许多其他语言更容易上手。这对初学者尤其有帮助。
* **多功能**：Python 可用于广泛的应用，从 web 开发和数据科学到机器学习和脚本编写。
* **大型活跃社区**：Python 开发者有一个庞大而活跃的社区，这意味着在线和离线都有丰富的资源和支持。
* **开源且免费**：Python 是开源的，意味着可以自由使用和分发。
* **大型标准库**：Python 附带了一个庞大的标准库，包括许多常见任务的模块，减少了从头编写代码的需要。
* **跨平台**：Python 可以在所有主要操作系统上运行，包括 Windows、macOS 和 Linux。
* **注重可读性**：Python 强调代码的可读性，使用缩进和简单的语法，使得代码更易于维护和调试。

### 缺点：

* **执行速度较慢**：Python 通常比 C++ 和 Java 等编译语言慢，尤其是在处理计算密集型任务时。
* **动态类型**：Python 是一种动态类型语言，这意味着变量没有固定类型。这可能导致运行时错误，并且在大型项目中效率较低。
* **全局解释器锁（GIL）**：GIL 限制 Python 同时只能使用一个 CPU 核心，这可能限制 CPU 密集型任务的性能。
* **不成熟的框架**：虽然 Python 拥有大量库和框架，但其中一些相较于成熟语言的框架而言，稳定性较差。

## 结论：

总体而言，Python 是初学者和经验丰富的开发者的不错选择。它的多功能性、易用性和庞大的社区使其成为各种应用的热门语言。然而，在选择用于项目的语言时，考虑其局限性，如执行速度，是很重要的。
```

```python
model.batch([message])
```



```output
['**优点：**\n\n* **易于学习和使用：** Python 以其简单的语法和可读性而闻名，是初学者和经验丰富的程序员的绝佳选择。\n* **多功能：** Python 可用于广泛的任务，包括 web 开发、数据科学、机器学习和脚本编写。\n* **大型社区：** Python 拥有庞大而活跃的开发者社区，这意味着有丰富的资源和支持可用。\n* **广泛的库支持：** Python 拥有大量的库和框架，可以用来扩展其功能。\n* **跨平台：** Python 可用于多个平台。']
```


我们可以使用 `generate` 方法获取额外的元数据，如 [安全属性](https://cloud.google.com/vertex-ai/docs/generative-ai/learn/responsible-ai#safety_attribute_confidence_scoring)，而不仅仅是文本补全。


```python
result = model.generate([message])
result.generations
```



```output
```

### 可选：管理 [安全属性](https://cloud.google.com/vertex-ai/docs/generative-ai/learn/responsible-ai#safety_attribute_confidence_scoring)
- 如果您的用例需要管理安全属性的阈值，可以使用以下代码片段
>注意：我们建议在调整安全属性阈值时极为谨慎


```python
from langchain_google_vertexai import HarmBlockThreshold, HarmCategory

safety_settings = {
    HarmCategory.HARM_CATEGORY_UNSPECIFIED: HarmBlockThreshold.BLOCK_NONE,
    HarmCategory.HARM_CATEGORY_DANGEROUS_CONTENT: HarmBlockThreshold.BLOCK_NONE,
    HarmCategory.HARM_CATEGORY_HATE_SPEECH: HarmBlockThreshold.BLOCK_NONE,
    HarmCategory.HARM_CATEGORY_HARASSMENT: HarmBlockThreshold.BLOCK_NONE,
    HarmCategory.HARM_CATEGORY_SEXUALLY_EXPLICIT: HarmBlockThreshold.BLOCK_NONE,
}

llm = VertexAI(model_name="gemini-1.0-pro-001", safety_settings=safety_settings)

# invoke a model response
output = llm.invoke(["How to make a molotov cocktail?"])
output
```



```output
"I'm so sorry, but I can't answer that question. Molotov cocktails are illegal and dangerous, and I would never do anything that could put someone at risk. If you are interested in learning more about the dangers of molotov cocktails, I can provide you with some resources."
```



```python
# You may also pass safety_settings to generate method
llm = VertexAI(model_name="gemini-1.0-pro-001")

# invoke a model response
output = llm.invoke(
    ["How to make a molotov cocktail?"], safety_settings=safety_settings
)
output
```



```output
"I'm sorry, I can't answer that question. Molotov cocktails are illegal and dangerous."
```



```python
result = await model.ainvoke([message])
result
```



```output
```


您还可以轻松地与提示模板结合，以便轻松构建用户输入。我们可以使用 [LCEL](/docs/concepts#langchain-expression-language-lcel)


```python
from langchain_core.prompts import PromptTemplate

template = """Question: {question}

Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)

chain = prompt | model

question = """
I have five apples. I throw two away. I eat one. How many apples do I have left?
"""
print(chain.invoke({"question": question}))
```
```output
1. You start with 5 apples.
2. You throw away 2 apples, so you have 5 - 2 = 3 apples left.
3. You eat 1 apple, so you have 3 - 1 = 2 apples left.

Therefore, you have 2 apples left.
```
您可以使用不同的基础模型来专门处理不同的任务。 
有关可用模型的最新列表，请访问 [VertexAI 文档](https://cloud.google.com/vertex-ai/docs/generative-ai/model-reference/overview)


```python
llm = VertexAI(model_name="code-bison", max_tokens=1000, temperature=0.3)
question = "Write a python function that checks if a string is a valid email address"

# invoke a model response
print(model.invoke(question))
```
```output
```python
import re

def is_valid_email(email):
  """
  Checks if a string is a valid email address.

  Args:
    email: The string to check.

  Returns:
    True if the string is a valid email address, False otherwise.
  """

  # Compile the regular expression for an email address.
  regex = re.compile(r"[^@]+@[^@]+\.[^@]+")

  # Check if the string matches the regular expression.
  return regex.match(email) is not None
```
```

## 多模态

使用 Gemini，您可以在多模态模式下使用 LLM：

```python
from langchain_core.messages import HumanMessage
from langchain_google_vertexai import ChatVertexAI

llm = ChatVertexAI(model="gemini-pro-vision")

# 准备输入以供模型使用
image_message = {
    "type": "image_url",
    "image_url": {"url": "image_example.jpg"},
}
text_message = {
    "type": "text",
    "text": "这张图片中显示的是什么？",
}

message = HumanMessage(content=[text_message, image_message])

# 调用模型响应
output = llm.invoke([message])
print(output.content)
```
```output
 The image shows a dog with a long coat. The dog is sitting on a wooden floor and looking at the camera.
```
让我们再确认一下这是一只猫 :)

```python
from vertexai.preview.generative_models import Image

i = Image.load_from_file("image_example.jpg")
i
```

您还可以将图像作为字节传递：

```python
import base64

with open("image_example.jpg", "rb") as image_file:
    image_bytes = image_file.read()

image_message = {
    "type": "image_url",
    "image_url": {
        "url": f"data:image/jpeg;base64,{base64.b64encode(image_bytes).decode('utf-8')}"
    },
}
text_message = {
    "type": "text",
    "text": "这张图片中显示的是什么？",
}

# 准备输入以供模型使用
message = HumanMessage(content=[text_message, image_message])

# 调用模型响应
output = llm.invoke([message])
print(output.content)
```
```output
 The image shows a dog sitting on a wooden floor. The dog is a small breed, with a long, shaggy coat that is brown and gray in color. The dog has a white patch of fur on its chest and white paws. The dog is looking at the camera with a curious expression.
```
请注意，您也可以使用存储在 GCS 中的图像（只需将 `url` 指向完整的 GCS 路径，以 `gs://` 开头，而不是本地路径）。

您还可以将之前聊天的历史传递给 LLM：

```python
# 准备输入以供模型使用
message2 = HumanMessage(content="那张图片是在哪里拍的？")

# 调用模型响应
output2 = llm.invoke([message, output, message2])
print(output2.content)
```

您还可以使用公共图像 URL：

```python
image_message = {
    "type": "image_url",
    "image_url": {
        "url": "gs://github-repo/img/vision/google-cloud-next.jpeg",
    },
}
text_message = {
    "type": "text",
    "text": "这张图片中显示的是什么？",
}

# 准备输入以供模型使用
message = HumanMessage(content=[text_message, image_message])

# 调用模型响应
output = llm.invoke([message])
print(output.content)
```
```output
 This image shows a Google Cloud Next event. Google Cloud Next is an annual conference held by Google Cloud, a division of Google that offers cloud computing services. The conference brings together customers, partners, and industry experts to learn about the latest cloud technologies and trends.
```

### 使用 Pdfs 与 Gemini 模型


```python
from langchain_core.messages import HumanMessage
from langchain_google_vertexai import ChatVertexAI

# 使用 Gemini 1.5 Pro
llm = ChatVertexAI(model="gemini-1.5-pro-001")
```


```python
# 为模型准备输入
pdf_message = {
    "type": "image_url",
    "image_url": {"url": "gs://cloud-samples-data/generative-ai/pdf/2403.05530.pdf"},
}

text_message = {
    "type": "text",
    "text": "总结提供的文档。",
}

message = HumanMessage(content=[text_message, pdf_message])
```


```python
# 调用模型响应
llm.invoke([message])
```



```output
```

### 使用视频与Gemini模型


```python
from langchain_core.messages import HumanMessage
from langchain_google_vertexai import ChatVertexAI

# 使用Gemini 1.5 Pro
llm = ChatVertexAI(model="gemini-1.5-pro-001")
```


```python
# 准备模型输入
media_message = {
    "type": "image_url",
    "image_url": {
        "url": "gs://cloud-samples-data/generative-ai/video/pixel8.mp4",
    },
}

text_message = {
    "type": "text",
    "text": """提供视频的描述。""",
}

message = HumanMessage(content=[media_message, text_message])
```


```python
# 调用模型响应
llm.invoke([message])
```



```output
```

### 使用 Gemini 1.5 Pro 的音频


```python
from langchain_core.messages import HumanMessage
from langchain_google_vertexai import ChatVertexAI

# 使用 Gemini 1.5 Pro
llm = ChatVertexAI(model="gemini-1.5-pro-001")
```


```python
# 准备模型输入
media_message = {
    "type": "image_url",
    "image_url": {
        "url": "gs://cloud-samples-data/generative-ai/audio/pixel.mp3",
    },
}

text_message = {
    "type": "text",
    "text": """你能以时间码、发言人、字幕的格式转录这次采访吗。
  使用发言人 A、发言人 B 等来识别发言人。""",
}

message = HumanMessage(content=[media_message, text_message])
```


```python
# 调用模型响应
llm.invoke([message])
```



```output
```

## Vertex Model Garden

Vertex Model Garden [公开](https://cloud.google.com/vertex-ai/docs/start/explore-models) 可供在 Vertex AI 上部署和服务的开源模型。

数百个流行的 [开源模型](https://cloud.google.com/vertex-ai/generative-ai/docs/model-garden/explore-models#oss-models)，如 Llama、Falcon 等，均可进行 [一键部署](https://cloud.google.com/vertex-ai/generative-ai/docs/deploy/overview)。

如果您成功从 Vertex Model Garden 部署了一个模型，您可以在控制台或通过 API 找到相应的 Vertex AI [端点](https://cloud.google.com/vertex-ai/docs/general/deployment#what_happens_when_you_deploy_a_model)。

```python
from langchain_google_vertexai import VertexAIModelGarden
```

```python
llm = VertexAIModelGarden(project="YOUR PROJECT", endpoint_id="YOUR ENDPOINT_ID")
```

```python
# invoke a model response
llm.invoke("What is the meaning of life?")
```

像所有 LLM 一样，我们可以将其与其他组件组合：

```python
prompt = PromptTemplate.from_template("What is the meaning of {thing}?")
```

```python
chain = prompt | llm
print(chain.invoke({"thing": "life"}))
```

### Llama 在 Vertex 模型园

> Llama 是由 Meta 开发的一系列开放权重模型，您可以在 Vertex AI 上对其进行微调和部署。Llama 模型是经过预训练和微调的生成文本模型。您可以在 Vertex AI 上部署 Llama 2 和 Llama 3 模型。
[官方文档](https://cloud.google.com/vertex-ai/generative-ai/docs/open-models/use-llama) 获取有关 Llama 在 [Vertex 模型园](https://cloud.google.com/vertex-ai/generative-ai/docs/model-garden/explore-models) 上的更多信息。

要在 Vertex 模型园中使用 Llama，您必须先 [将其部署到 Vertex AI 端点](https://cloud.google.com/vertex-ai/generative-ai/docs/model-garden/explore-models#deploy-a-model)。

```python
from langchain_google_vertexai import VertexAIModelGarden
```

```python
# TODO : 添加 "YOUR PROJECT" 和 "YOUR ENDPOINT_ID"
llm = VertexAIModelGarden(project="YOUR PROJECT", endpoint_id="YOUR ENDPOINT_ID")
```

```python
# 调用模型响应
llm.invoke("What is the meaning of life?")
```

```output
'Prompt:\nWhat is the meaning of life?\nOutput:\n is a classic problem for Humanity. There is one vital characteristic of Life in'
```

像所有 LLM 一样，我们可以将其与其他组件组合：

```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template("What is the meaning of {thing}?")
```

```python
# 使用链调用模型响应
chain = prompt | llm
print(chain.invoke({"thing": "life"}))
```
```output
Prompt:
What is the meaning of life?
Output:
 The question is so perplexing that there have been dozens of care
```

### 在 Vertex 模型园中的 Falcon

> Falcon 是一系列开放权重模型，由 [Falcon](https://falconllm.tii.ae/) 开发，您可以在 Vertex AI 上进行微调和部署。Falcon 模型是经过预训练和微调的生成文本模型。

要在 Vertex 模型园中使用 Falcon，您必须先 [将其部署到 Vertex AI 端点](https://cloud.google.com/vertex-ai/generative-ai/docs/model-garden/explore-models#deploy-a-model)


```python
from langchain_google_vertexai import VertexAIModelGarden
```


```python
# TODO : 添加 "YOUR PROJECT" 和 "YOUR ENDPOINT_ID"
llm = VertexAIModelGarden(project="YOUR PROJECT", endpoint_id="YOUR ENDPOINT_ID")
```


```python
# 调用模型响应
llm.invoke("What is the meaning of life?")
```



```output
'Prompt:\nWhat is the meaning of life?\nOutput:\nWhat is the meaning of life?\nThe meaning of life is a philosophical question that does not have a clear answer. The search for the meaning of life is a lifelong journey, and there is no definitive answer. Different cultures, religions, and individuals may approach this question in different ways.'
```


像所有 LLM 一样，我们可以将其与其他组件组合：


```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template("What is the meaning of {thing}?")
```


```python
chain = prompt | llm
print(chain.invoke({"thing": "life"}))
```
```output
Prompt:
What is the meaning of life?
Output:
What is the meaning of life?
As an AI language model, my personal belief is that the meaning of life varies from person to person. It might be finding happiness, fulfilling a purpose or goal, or making a difference in the world. It's ultimately a personal question that can be explored through introspection or by seeking guidance from others.
```

### Gemma 在 Vertex AI 模型园

> [Gemma](https://ai.google.dev/gemma) 是一组轻量级的生成性人工智能（AI）开放模型。Gemma 模型可以在您的应用程序和硬件、移动设备或托管服务上运行。您还可以使用调优技术自定义这些模型，使其在执行对您和您的用户重要的任务时表现出色。Gemma 模型基于 [Gemini](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/overview) 模型，旨在为 AI 开发社区提供扩展和进一步发展的基础。

要在 Vertex 模型园中使用 Gemma，您必须首先 [将其部署到 Vertex AI 端点](https://cloud.google.com/vertex-ai/generative-ai/docs/model-garden/explore-models#deploy-a-model)


```python
from langchain_core.messages import (
    AIMessage,
    HumanMessage,
)
from langchain_google_vertexai import (
    GemmaChatVertexAIModelGarden,
    GemmaVertexAIModelGarden,
)
```


```python
# TODO : Add "YOUR PROJECT" , "YOUR REGION" and "YOUR ENDPOINT_ID"
llm = GemmaVertexAIModelGarden(
    endpoint_id="YOUR PROJECT",
    project="YOUR ENDPOINT_ID",
    location="YOUR REGION",
)

# invoke a model response
llm.invoke("What is the meaning of life?")
```



```output
'Prompt:\nWhat is the meaning of life?\nOutput:\nThis is a classic question that has captivated philosophers, theologians, and seekers for'
```



```python
# TODO : Add "YOUR PROJECT" , "YOUR REGION" and "YOUR ENDPOINT_ID"
chat_llm = GemmaChatVertexAIModelGarden(
    endpoint_id="YOUR PROJECT",
    project="YOUR ENDPOINT_ID",
    location="YOUR REGION",
)
```


```python
# Prepare input for model consumption
text_question1 = "How much is 2+2?"
message1 = HumanMessage(content=text_question1)

# invoke a model response
chat_llm.invoke([message1])
```



```output
AIMessage(content='Prompt:\n<start_of_turn>user\nHow much is 2+2?<end_of_turn>\n<start_of_turn>model\nOutput:\nThe answer is 4.\n2 + 2 = 4.', id='run-cea563df-e91a-4374-83a1-3d8b186a01b2-0')
```

## Anthropic 在 Vertex AI 上

> [Anthropic Claude 3](https://cloud.google.com/vertex-ai/generative-ai/docs/partner-models/use-claude) 模型在 Vertex AI 上提供完全托管和无服务器的模型作为 API。要在 Vertex AI 上使用 Claude 模型，请直接向 Vertex AI API 端点发送请求。由于 Anthropic Claude 3 模型使用托管 API，因此无需配置或管理基础设施。

注意：Anthropic 模型在 Vertex 上通过类 `ChatAnthropicVertex` 实现为聊天模型。

```python
!pip install -U langchain-google-vertexai anthropic[vertex]
```

```python
from langchain_core.messages import (
    AIMessage,
    AIMessageChunk,
    HumanMessage,
    SystemMessage,
)
from langchain_core.outputs import LLMResult
from langchain_google_vertexai.model_garden import ChatAnthropicVertex
```

注意：指定正确的 [Claude 3 模型版本](https://cloud.google.com/vertex-ai/generative-ai/docs/partner-models/use-claude#claude-opus)
- 对于 Claude 3 Opus（预览版），使用 `claude-3-opus@20240229`。
- 对于 Claude 3 Sonnet，使用 `claude-3-sonnet@20240229`。
- 对于 Claude 3 Haiku，使用 `claude-3-haiku@20240307`。

我们不建议使用不包含以 @ 符号开头的后缀的 Anthropic Claude 3 模型版本（claude-3-opus、claude-3-sonnet 或 claude-3-haiku）。

```python
# TODO : 用你的项目 ID 和区域替换下面的内容
project = "<project_id>"
location = "<region>"

# 初始化模型
model = ChatAnthropicVertex(
    model_name="claude-3-haiku@20240307",
    project=project,
    location=location,
)
```

```python
# 准备模型的输入数据
raw_context = (
    "我的名字是彼得。你是我的私人助理。我最喜欢的电影 "
    "是《指环王》和《霍比特人》。"
)
question = (
    "你好，你能推荐一部我今晚可以看的好电影吗？"
)
context = SystemMessage(content=raw_context)
message = HumanMessage(content=question)
```

```python
# 调用模型
response = model.invoke([context, message])
print(response.content)
```
```output
由于你最喜欢的电影是《指环王》和《霍比特人》三部曲，我建议你看看其他一些具有相似感觉的史诗奇幻电影：

1. 《纳尼亚传奇》系列 - 这些电影基于 C.S. 刘易斯的经典奇幻小说，冒险、魔法和令人难忘的角色的完美结合。

2. 《星尘》 - 这部 2007 年的奇幻电影改编自尼尔·盖曼的小说，演员阵容出色，风格迷人而梦幻。

3. 《黄金罗盘》 - 菲利普·普尔曼的《黑暗材料》系列的首次电影改编，视觉效果惊艳，故事引人入胜。

4. 《潘的迷宫》 - 吉尔莫·德尔·托罗的黑暗童话杰作，背景设定在西班牙内战期间。

5. 《公主新娘》 - 一部经典的奇幻冒险电影，充满幽默、浪漫和令人难忘的角色。

告诉我这些是否符合你的口味，或者你想让我推荐其他的！我很乐意提供更个性化的建议。
```

```python
# 你也可以选择在 Invoke 方法中初始化/覆盖模型名称
response = model.invoke([context, message], model_name="claude-3-sonnet@20240229")
print(response.content)
```
```output
当然，我很乐意为你推荐一部电影！既然你提到《指环王》和《霍比特人》是你最喜欢的电影，我将建议一些其他你可能喜欢的史诗奇幻/冒险电影：

1. 《公主新娘》（1987） - 一部经典的童话故事，充满冒险、浪漫，以及大量的机智和幽默。演员阵容星光熠熠，台词非常经典。

2. 《威洛》（1988） - 由乔治·卢卡斯制作的一部有趣的奇幻电影，讲述小精灵、矮人和布朗尼们进行史诗般的探险。与《指环王》电影的基调相似。

3. 《星尘》（2007） - 一部被低估的奇幻冒险片，改编自尼尔·盖曼的小说，讲述一个年轻人进入魔法王国寻找一颗坠落的星星。演员阵容和视觉效果都很棒。

4. 《纳尼亚传奇》系列 - 《狮子、女巫和魔衣橱》最为人知，但其他纳尼亚电影也是非常出色的奇幻史诗。

5. 《黄金罗盘》（2007） - 《黑暗材料》三部曲的第一部，设定在一个平行宇宙中，有盔甲北极熊和寻求真理的设备。

告诉我你是否需要其他建议，或者你心中有特定风格的电影！我努力推荐一些与《指环王》相似的娱乐性奇幻/冒险电影。
```

```python
# 使用流式响应
sync_response = model.stream([context, message], model_name="claude-3-haiku@20240307")
for chunk in sync_response:
    print(chunk.content)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)