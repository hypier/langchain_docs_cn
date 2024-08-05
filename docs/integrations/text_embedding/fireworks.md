---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/fireworks.ipynb
---

# FireworksEmbeddings

本笔记本解释了如何使用包含在langchain_fireworks包中的Fireworks Embeddings来在langchain中嵌入文本。我们在这个例子中使用默认的nomic-ai v1.5模型。


```python
%pip install -qU langchain-fireworks
```

## 设置


```python
from langchain_fireworks import FireworksEmbeddings
```


```python
import getpass
import os

if "FIREWORKS_API_KEY" not in os.environ:
    os.environ["FIREWORKS_API_KEY"] = getpass.getpass("Fireworks API Key:")
```

# 使用嵌入模型
通过 `FireworksEmbeddings`，您可以直接使用默认模型 'nomic-ai/nomic-embed-text-v1.5'，或者如果有可用的其他模型则可以设置不同的模型。

```python
embedding = FireworksEmbeddings(model="nomic-ai/nomic-embed-text-v1.5")
```

```python
res_query = embedding.embed_query("The test information")
res_document = embedding.embed_documents(["test1", "another test"])
print(res_query[:5])
print(res_document[1][:5])
```
```output
[0.01367950439453125, 0.0103607177734375, -0.157958984375, -0.003070831298828125, 0.05926513671875]
[0.0369873046875, 0.00545501708984375, -0.179931640625, -0.018707275390625, 0.0552978515625]
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)