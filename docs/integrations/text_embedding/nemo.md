---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/nemo.ipynb
---

# NVIDIA NeMo 嵌入

使用 `NeMoEmbeddings` 类连接到 NVIDIA 的嵌入服务。

NeMo 检索嵌入微服务 (NREM) 将最先进的文本嵌入能力带入您的应用程序，提供无与伦比的自然语言处理和理解能力。无论您是在开发语义搜索、检索增强生成 (RAG) 流水线，还是任何需要使用文本嵌入的应用程序，NREM 都能满足您的需求。NREM 构建在 NVIDIA 软件平台之上，结合了 CUDA、TensorRT 和 Triton，提供最先进的 GPU 加速文本嵌入模型服务。

NREM 使用 NVIDIA 的 TensorRT 构建在 Triton 推理服务器之上，以优化文本嵌入模型的推理。

## 导入


```python
from langchain_community.embeddings import NeMoEmbeddings
```

## 设置


```python
batch_size = 16
model = "NV-Embed-QA-003"
api_endpoint_url = "http://localhost:8080/v1/embeddings"
```


```python
embedding_model = NeMoEmbeddings(
    batch_size=batch_size, model=model, api_endpoint_url=api_endpoint_url
)
```
```output
检查端点是否可用: http://localhost:8080/v1/embeddings
```

```python
embedding_model.embed_query("This is a test.")
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)