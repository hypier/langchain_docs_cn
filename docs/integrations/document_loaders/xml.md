---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/xml.ipynb
---

# XML

`UnstructuredXMLLoader` 用于加载 `XML` 文件。加载器处理 `.xml` 文件。页面内容将是从 XML 标签中提取的文本。


```python
from langchain_community.document_loaders import UnstructuredXMLLoader

loader = UnstructuredXMLLoader(
    "./example_data/factbook.xml",
)
docs = loader.load()
docs[0]
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)