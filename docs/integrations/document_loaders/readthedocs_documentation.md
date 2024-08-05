---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/readthedocs_documentation.ipynb
---

# ReadTheDocs 文档

>[Read the Docs](https://readthedocs.org/) 是一个开源的免费软件文档托管平台。它生成使用 `Sphinx` 文档生成器编写的文档。

本笔记本涵盖了如何加载作为 `Read-The-Docs` 构建的一部分生成的 HTML 内容。

在实际应用中的示例，请参见 [这里](https://github.com/langchain-ai/chat-langchain)。

这假设 HTML 已经被抓取到一个文件夹中。可以通过取消注释并运行以下命令来完成此操作


```python
%pip install --upgrade --quiet  beautifulsoup4
```


```python
#!wget -r -A.html -P rtdocs https://python.langchain.com/en/latest/
```


```python
from langchain_community.document_loaders import ReadTheDocsLoader
```


```python
loader = ReadTheDocsLoader("rtdocs", features="html.parser")
```


```python
docs = loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)