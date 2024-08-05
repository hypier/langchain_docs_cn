---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/fastembed.ipynb
---

# FastEmbed by Qdrant

>[FastEmbed](https://qdrant.github.io/fastembed/) 来自 [Qdrant](https://qdrant.tech)，是一个轻量级、快速的 Python 库，用于生成嵌入。 
>
>- 量化模型权重
>- ONNX Runtime，无需 PyTorch 依赖
>- CPU 优先设计
>- 数据并行性，适用于大规模数据集的编码。

## 依赖项

要在 LangChain 中使用 FastEmbed，请安装 `fastembed` Python 包。

```python
%pip install --upgrade --quiet  fastembed
```

## 导入


```python
from langchain_community.embeddings.fastembed import FastEmbedEmbeddings
```

## 实例化 FastEmbed

### 参数
- `model_name: str` (默认: "BAAI/bge-small-en-v1.5")
    > 要使用的 FastEmbedding 模型名称。您可以在 [这里](https://qdrant.github.io/fastembed/examples/Supported_Models/) 找到支持的模型列表。

- `max_length: int` (默认: 512)
    > 最大令牌数。对于大于 512 的值，未知行为。

- `cache_dir: Optional[str]` (默认: None)
    > 缓存目录的路径。默认为父目录中的 `local_cache`。

- `threads: Optional[int]` (默认: None)
    > 单个 onnxruntime 会话可以使用的线程数。

- `doc_embed_type: Literal["default", "passage"]` (默认: "default")
    > "default": 使用 FastEmbed 的默认嵌入方法。
    
    > "passage": 在嵌入之前用 "passage" 前缀文本。

- `batch_size: int` (默认: 256)
    > 编码的批量大小。更高的值会使用更多内存，但速度更快。

- `parallel: Optional[int]` (默认: None)

    > 如果 `>1`，将使用数据并行编码，建议用于大数据集的离线编码。
    > 如果 `0`，使用所有可用核心。
    > 如果 `None`，不使用数据并行处理，而是使用默认的 onnxruntime 线程。

```python
embeddings = FastEmbedEmbeddings()
```

## 用法

### 生成文档嵌入


```python
document_embeddings = embeddings.embed_documents(
    ["This is a document", "This is some other document"]
)
```

### 生成查询嵌入


```python
query_embeddings = embeddings.embed_query("This is a query")
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)