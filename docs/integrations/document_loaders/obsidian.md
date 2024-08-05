---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/obsidian.ipynb
---

# Obsidian

>[Obsidian](https://obsidian.md/) 是一个强大且可扩展的知识库，基于您本地的纯文本文件夹工作。

本笔记本涵盖如何从 `Obsidian` 数据库加载文档。

由于 `Obsidian` 只是以 Markdown 文件的文件夹形式存储在磁盘上，因此加载器只需提供该目录的路径。

`Obsidian` 文件有时还包含 [metadata](https://help.obsidian.md/Editing+and+formatting/Metadata)，这是文件顶部的 YAML 块。这些值将被添加到文档的元数据中。（`ObsidianLoader` 也可以传递 `collect_metadata=False` 参数以禁用此行为。）

```python
from langchain_community.document_loaders import ObsidianLoader
```

```python
loader = ObsidianLoader("<path-to-obsidian>")
```

```python
docs = loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)