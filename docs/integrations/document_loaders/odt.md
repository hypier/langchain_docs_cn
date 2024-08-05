---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/odt.ipynb
---

# 开放文档格式 (ODT)

> [开放文档格式用于办公应用程序 (ODF)](https://en.wikipedia.org/wiki/OpenDocument)，也称为 `OpenDocument`，是一种用于文字处理文档、电子表格、演示文稿和图形的开放文件格式，采用 ZIP 压缩的 XML 文件。它的开发旨在为办公应用程序提供一种开放的、基于 XML 的文件格式规范。

> 该标准由结构化信息标准促进组织 (`OASIS`) 联盟的技术委员会开发和维护。它基于 Sun Microsystems 为 OpenOffice.org XML 提供的规范，后者是 `OpenOffice.org` 和 `LibreOffice` 的默认格式。它最初是为 `StarOffice` 开发的，目的是“为办公文档提供开放标准”。

`UnstructuredODTLoader` 用于加载 `Open Office ODT` 文件。


```python
from langchain_community.document_loaders import UnstructuredODTLoader

loader = UnstructuredODTLoader("example_data/fake.odt", mode="elements")
docs = loader.load()
docs[0]
```



```output
Document(page_content='Lorem ipsum dolor sit amet.', metadata={'source': 'example_data/fake.odt', 'category_depth': 0, 'file_directory': 'example_data', 'filename': 'fake.odt', 'last_modified': '2023-12-19T13:42:18', 'languages': ['por', 'cat'], 'filetype': 'application/vnd.oasis.opendocument.text', 'category': 'Title'})
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)