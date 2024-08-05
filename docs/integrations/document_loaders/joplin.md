---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/joplin.ipynb
---

# Joplin

>[Joplin](https://joplinapp.org/) 是一个开源笔记应用程序。捕捉您的想法，并从任何设备安全访问它们。

本笔记本涵盖如何从 `Joplin` 数据库加载文档。

`Joplin` 提供了一个 [REST API](https://joplinapp.org/api/references/rest_api/) 用于访问其本地数据库。此加载器使用 API 检索数据库中的所有笔记及其元数据。这需要一个访问令牌，可以通过以下步骤从应用程序获取：

1. 打开 `Joplin` 应用程序。在加载文档时，应用程序必须保持打开状态。
2. 转到设置 / 选项并选择“网页剪辑器”。
3. 确保已启用网页剪辑器服务。
4. 在“高级选项”下，复制授权令牌。

您可以直接用访问令牌初始化加载器，或者将其存储在环境变量 JOPLIN_ACCESS_TOKEN 中。

另一种方法是将 `Joplin` 的笔记数据库导出为 Markdown 文件（可选地，带有 Front Matter 元数据），并使用 Markdown 加载器，例如 ObsidianLoader，来加载它们。


```python
from langchain_community.document_loaders import JoplinLoader
```


```python
loader = JoplinLoader(access_token="<access-token>")
```


```python
docs = loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)