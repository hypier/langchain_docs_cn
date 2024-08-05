---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/notion.ipynb
---

# Notion DB 1/2

>[Notion](https://www.notion.so/) 是一个协作平台，支持修改过的 Markdown，集成了看板、任务、维基和数据库。它是一个用于笔记、知识和数据管理，以及项目和任务管理的全能工作空间。

本笔记本涵盖了如何从 Notion 数据库导出中加载文档。

要获取这个 Notion 导出，请按照以下说明操作：

## 🧑 导入您自己的数据集的说明

从 Notion 导出您的数据集。您可以通过点击右上角的三个点，然后点击 `Export` 来完成此操作。

导出时，请确保选择 `Markdown & CSV` 格式选项。

这将生成一个 `.zip` 文件在您的下载文件夹中。将 `.zip` 文件移动到此存储库中。

运行以下命令以解压缩 zip 文件（根据需要将 `Export...` 替换为您自己的文件名）。

```shell
unzip Export-d3adfe0f-3131-4bf3-8987-a52017fc1bae.zip -d Notion_DB
```

运行以下命令以导入数据。

```python
from langchain_community.document_loaders import NotionDirectoryLoader
```


```python
loader = NotionDirectoryLoader("Notion_DB")
```


```python
docs = loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)