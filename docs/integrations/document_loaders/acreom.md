---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/acreom.ipynb
---

# acreom

[acreom](https://acreom.com) 是一个以开发者为中心的知识库，任务在本地的 markdown 文件上运行。

以下是如何将本地的 acreom vault 加载到 Langchain 的示例。由于 acreom 中的本地 vault 是一组纯文本的 .md 文件，因此加载器需要目录的路径。

Vault 文件可能包含一些以 YAML 头部存储的元数据。如果 `collect_metadata` 设置为 true，这些值将被添加到文档的元数据中。 


```python
from langchain_community.document_loaders import AcreomLoader
```


```python
loader = AcreomLoader("<path-to-acreom-vault>", collect_metadata=False)
```


```python
docs = loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)