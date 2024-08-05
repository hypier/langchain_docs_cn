---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/mistralai.ipynb
---

# MistralAI

本笔记本解释了如何使用包含在 langchain_mistralai 包中的 MistralAIEmbeddings 来嵌入 langchain 中的文本。

```python
# pip install -U langchain-mistralai
```

## 导入库


```python
from langchain_mistralai import MistralAIEmbeddings
```


```python
embedding = MistralAIEmbeddings(api_key="your-api-key")
```

# 使用嵌入模型
使用 `MistralAIEmbeddings`，您可以直接使用默认模型 'mistral-embed'，或者如果有可用的模型，可以设置为其他模型。

```python
embedding.model = "mistral-embed"  # or your preferred model if available
```

```python
res_query = embedding.embed_query("The test information")
res_document = embedding.embed_documents(["test1", "another test"])
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)