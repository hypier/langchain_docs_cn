---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/upstage.ipynb
---

# Upstage

[Upstage](https://upstage.ai) 是一家领先的人工智能 (AI) 公司，专注于提供超越人类水平的 LLM 组件性能。

## Solar LLM

**Solar Mini Chat** 是一个快速而强大的高级大型语言模型，专注于英语和韩语。它经过特别调优，旨在多轮对话场景中表现出色，显示出在各种自然语言处理任务中的增强性能，如多轮对话或需要理解长上下文的任务，例如 RAG（检索增强生成），与其他相似规模的模型相比。这样的调优使其能够更有效地处理更长的对话，特别适合互动应用。

除了 Solar，Upstage 还提供现实世界 RAG（检索增强生成）的功能，如 **Groundedness Check** 和 **Layout Analysis**。

## 安装与设置

安装 `langchain-upstage` 包：

```bash
pip install -qU langchain-core langchain-upstage
```

获取 [API 密钥](https://console.upstage.ai) 并设置环境变量 `UPSTAGE_API_KEY`。

## Upstage LangChain 集成

| API | 描述 | 导入 | 示例用法 |
| --- | --- | --- | --- |
| Chat | 使用 Solar Mini Chat 构建助手 | `from langchain_upstage import ChatUpstage` | [Go](../../chat/upstage) |
| Text Embedding | 将字符串嵌入为向量 | `from langchain_upstage import UpstageEmbeddings` | [Go](../../text_embedding/upstage) |
| Groundedness Check | 验证助手响应的基础性 | `from langchain_upstage import UpstageGroundednessCheck` | [Go](../../tools/upstage_groundedness_check) |
| Layout Analysis | 序列化包含表格和图形的文档 | `from langchain_upstage import UpstageLayoutAnalysisLoader` | [Go](../../document_loaders/upstage) |

有关功能的更多详细信息，请参见 [文档](https://developers.upstage.ai/)。

## 快速示例

### 环境设置


```python
import os

os.environ["UPSTAGE_API_KEY"] = "YOUR_API_KEY"
```

### 聊天



```python
from langchain_upstage import ChatUpstage

chat = ChatUpstage()
response = chat.invoke("Hello, how are you?")
print(response)
```

### 文本嵌入




```python
from langchain_upstage import UpstageEmbeddings

embeddings = UpstageEmbeddings(model="solar-embedding-1-large")
doc_result = embeddings.embed_documents(
    ["Sung is a professor.", "This is another document"]
)
print(doc_result)

query_result = embeddings.embed_query("What does Sung do?")
print(query_result)
```

### 可靠性检查


```python
from langchain_upstage import UpstageGroundednessCheck

groundedness_check = UpstageGroundednessCheck()

request_input = {
    "context": "Mauna Kea is an inactive volcano on the island of Hawaii. Its peak is 4,207.3 m above sea level, making it the highest point in Hawaii and second-highest peak of an island on Earth.",
    "answer": "Mauna Kea is 5,207.3 meters tall.",
}
response = groundedness_check.invoke(request_input)
print(response)
```

### 布局分析


```python
from langchain_upstage import UpstageLayoutAnalysisLoader

file_path = "/PATH/TO/YOUR/FILE.pdf"
layzer = UpstageLayoutAnalysisLoader(file_path, split="page")

# 为了提高内存效率，考虑使用 lazy_load 方法逐页加载文档。
docs = layzer.load()  # 或 layzer.lazy_load()

for doc in docs[:3]:
    print(doc)
```