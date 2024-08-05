---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/nvidia_ai_endpoints.ipynb
---

# NVIDIA NIMs 

`langchain-nvidia-ai-endpoints` 包含 LangChain 集成，构建与 NVIDIA NIM 推理微服务上的模型的应用程序。NIM 支持来自社区和 NVIDIA 的聊天、嵌入和重新排序模型等多个领域的模型。这些模型经过 NVIDIA 优化，以在 NVIDIA 加速基础设施上提供最佳性能，并作为 NIM 部署，NIM 是一个易于使用的预构建容器，可以通过在 NVIDIA 加速基础设施上使用单个命令随处部署。

NVIDIA 托管的 NIM 部署可在 [NVIDIA API 目录](https://build.nvidia.com/) 上进行测试。测试后，NIM 可以使用 NVIDIA AI Enterprise 许可证从 NVIDIA 的 API 目录导出，并在本地或云中运行，使企业拥有其知识产权和 AI 应用程序的所有权和完全控制权。

NIM 以每个模型为基础打包为容器镜像，并通过 NVIDIA NGC 目录作为 NGC 容器镜像分发。NIM 的核心提供了简单、一致和熟悉的 API，以便在 AI 模型上运行推理。

本示例介绍了如何使用 LangChain 与支持的 [NVIDIA Retrieval QA Embedding Model](https://build.nvidia.com/nvidia/embed-qa-4) 进行交互，以实现通过 `NVIDIAEmbeddings` 类进行的 [检索增强生成](https://developer.nvidia.com/blog/build-enterprise-retrieval-augmented-generation-apps-with-nvidia-retrieval-qa-embedding-model/)。

有关通过此 API 访问聊天模型的更多信息，请查看 [ChatNVIDIA](https://python.langchain.com/docs/integrations/chat/nvidia_ai_endpoints/) 文档。

## 安装


```python
%pip install --upgrade --quiet  langchain-nvidia-ai-endpoints
```

## 设置

**开始之前：**

1. 创建一个免费的[NVIDIA](https://build.nvidia.com/)账户，该平台托管NVIDIA AI Foundation模型。

2. 选择`Retrieval`选项卡，然后选择您选择的模型。

3. 在`Input`下选择`Python`选项卡，然后点击`Get API Key`。接着点击`Generate Key`。

4. 复制并保存生成的密钥为`NVIDIA_API_KEY`。从此，您应该可以访问这些端点。

```python
import getpass
import os

# del os.environ['NVIDIA_API_KEY']  ## delete key and reset
if os.environ.get("NVIDIA_API_KEY", "").startswith("nvapi-"):
    print("Valid NVIDIA_API_KEY already in environment. Delete to reset")
else:
    nvapi_key = getpass.getpass("NVAPI Key (starts with nvapi-): ")
    assert nvapi_key.startswith("nvapi-"), f"{nvapi_key[:5]}... is not a valid key"
    os.environ["NVIDIA_API_KEY"] = nvapi_key
```

我们应该能够在该列表中看到一个嵌入模型，可以与LLM结合使用，以实现有效的RAG解决方案。我们可以通过`NVIDIAEmbeddings`类与该模型以及NIM支持的其他嵌入模型进行接口。

## 在NVIDIA API目录中使用NIM

在初始化嵌入模型时，可以通过传递模型名称（例如下面的`NV-Embed-QA`）来选择模型，或者不传递任何参数以使用默认值。

```python
from langchain_nvidia_ai_endpoints import NVIDIAEmbeddings

embedder = NVIDIAEmbeddings(model="NV-Embed-QA")
```

该模型是经过微调的E5-large模型，支持预期的`Embeddings`方法，包括：

- `embed_query`: 为查询样本生成查询嵌入。

- `embed_documents`: 为要搜索的文档列表生成段落嵌入。

- `aembed_query`/`aembed_documents`: 上述方法的异步版本。

## 使用自托管的 NVIDIA NIM
准备好部署时，您可以使用 NVIDIA NIM 自托管模型——它包含在 NVIDIA AI Enterprise 软件许可证中——并在任何地方运行它们，从而拥有自定义的所有权和对您的知识产权 (IP) 及 AI 应用程序的完全控制。

[了解有关 NIM 的更多信息](https://developer.nvidia.com/blog/nvidia-nim-offers-optimized-inference-microservices-for-deploying-ai-models-at-scale/)



```python
from langchain_nvidia_ai_endpoints import NVIDIAEmbeddings

# connect to an embedding NIM running at localhost:8080
embedder = NVIDIAEmbeddings(base_url="http://localhost:8080/v1")
```

### **相似性**

以下是对这些数据点相似性的快速测试：

**查询：**

- 科姆恰特卡的天气怎么样？

- 意大利以什么食物闻名？

- 我叫什么名字？我打赌你不记得...

- 生活的意义到底是什么？

- 生活的意义就是享受乐趣 :D

**文档：**

- 科姆恰特卡的天气寒冷，冬季漫长而严酷。

- 意大利以意大利面、比萨饼、冰淇淋和浓缩咖啡而闻名。

- 我无法记住个人名字，只提供信息。

- 生活的目的因人而异，通常被视为个人的成就感。

- 享受生活的每一刻确实是一种美好的方式。

### 嵌入运行时


```python
print("\nSequential Embedding: ")
q_embeddings = [
    embedder.embed_query("What's the weather like in Komchatka?"),
    embedder.embed_query("What kinds of food is Italy known for?"),
    embedder.embed_query("What's my name? I bet you don't remember..."),
    embedder.embed_query("What's the point of life anyways?"),
    embedder.embed_query("The point of life is to have fun :D"),
]
print("Shape:", (len(q_embeddings), len(q_embeddings[0])))
```

### 文档嵌入


```python
print("\nBatch Document Embedding: ")
d_embeddings = embedder.embed_documents(
    [
        "Komchatka's weather is cold, with long, severe winters.",
        "Italy is famous for pasta, pizza, gelato, and espresso.",
        "I can't recall personal names, only provide information.",
        "Life's purpose varies, often seen as personal fulfillment.",
        "Enjoying life's moments is indeed a wonderful approach.",
    ]
)
print("Shape:", (len(q_embeddings), len(q_embeddings[0])))
```

现在我们已经生成了嵌入向量，可以对结果进行简单的相似性检查，以查看哪些文档会在检索任务中触发为合理的答案：


```python
%pip install --upgrade --quiet  matplotlib scikit-learn
```


```python
import matplotlib.pyplot as plt
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

# Compute the similarity matrix between q_embeddings and d_embeddings
cross_similarity_matrix = cosine_similarity(
    np.array(q_embeddings),
    np.array(d_embeddings),
)

# Plotting the cross-similarity matrix
plt.figure(figsize=(8, 6))
plt.imshow(cross_similarity_matrix, cmap="Greens", interpolation="nearest")
plt.colorbar()
plt.title("Cross-Similarity Matrix")
plt.xlabel("Query Embeddings")
plt.ylabel("Document Embeddings")
plt.grid(True)
plt.show()
```

作为提醒，发送到我们系统的查询和文档是：

**查询：**

- Komchatka的天气怎么样？

- 意大利以什么食物闻名？

- 我叫什么名字？我敢打赌你不记得...

- 生活的意义是什么？

- 生活的意义就是享受乐趣 :D

**文档：**

- Komchatka's weather is cold, with long, severe winters.

- Italy is famous for pasta, pizza, gelato, and espresso.

- I can't recall personal names, only provide information.

- Life's purpose varies, often seen as personal fulfillment.

- Enjoying life's moments is indeed a wonderful approach.

## 截断

嵌入模型通常具有固定的上下文窗口，决定了可以嵌入的最大输入令牌数量。这个限制可能是一个硬性限制，等于模型的最大输入令牌长度，或是一个有效限制，超出该限制后嵌入的准确性会下降。

由于模型在令牌上操作，而应用程序通常处理文本，因此应用程序确保其输入保持在模型的令牌限制内可能会很具挑战性。默认情况下，如果输入过大，会抛出异常。

为此，NVIDIA 的 NIMs（API 目录或本地）提供了一个 `truncate` 参数，如果输入过大，可以在服务器端截断输入。

`truncate` 参数有三个选项：
 - "NONE": 默认选项。如果输入过大，会抛出异常。
 - "START": 服务器从开始（左侧）截断输入，必要时丢弃令牌。
 - "END": 服务器从结束（右侧）截断输入，必要时丢弃令牌。


```python
long_text = "AI is amazing, amazing is " * 100
```


```python
strict_embedder = NVIDIAEmbeddings()
try:
    strict_embedder.embed_query(long_text)
except Exception as e:
    print("Error:", e)
```


```python
truncating_embedder = NVIDIAEmbeddings(truncate="END")
truncating_embedder.embed_query(long_text)[:5]
```

## RAG 检索：

以下是对 [LangChain 表达语言检索食谱条目](https://python.langchain.com/docs/expression_language/cookbook/retrieval) 初始示例的重新利用，但使用的是 AI Foundation Models 的 [Mixtral 8x7B Instruct](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/ai-foundation/models/mixtral-8x7b) 和 [NVIDIA Retrieval QA Embedding](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/ai-foundation/models/nvolve-40k) 模型，这些模型在他们的游乐场环境中可用。食谱中的后续示例也按预期运行，我们鼓励您使用这些选项进行探索。

**提示：** 我们建议将 Mixtral 用于内部推理（即数据提取、工具选择等的指令跟随），而将 Llama-Chat 用于基于历史和上下文为此用户提供单一最终“总结性简单响应”的响应。

```python
%pip install --upgrade --quiet  langchain faiss-cpu tiktoken langchain_community

from operator import itemgetter

from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_nvidia_ai_endpoints import ChatNVIDIA
```


```python
vectorstore = FAISS.from_texts(
    ["harrison worked at kensho"],
    embedding=NVIDIAEmbeddings(model="NV-Embed-QA"),
)
retriever = vectorstore.as_retriever()

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "Answer solely based on the following context:\n<Documents>\n{context}\n</Documents>",
        ),
        ("user", "{question}"),
    ]
)

model = ChatNVIDIA(model="ai-mixtral-8x7b-instruct")

chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

chain.invoke("where did harrison work?")
```


```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "Answer using information solely based on the following context:\n<Documents>\n{context}\n</Documents>"
            "\nSpeak only in the following language: {language}",
        ),
        ("user", "{question}"),
    ]
)

chain = (
    {
        "context": itemgetter("question") | retriever,
        "question": itemgetter("question"),
        "language": itemgetter("language"),
    }
    | prompt
    | model
    | StrOutputParser()
)

chain.invoke({"question": "where did harrison work", "language": "italian"})
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)