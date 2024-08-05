---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/epub.ipynb
---

# EPub 

>[EPUB](https://en.wikipedia.org/wiki/EPUB) 是一种电子书文件格式，使用 ".epub" 文件扩展名。这个术语是电子出版物的缩写，有时被写作 ePub。`EPUB` 得到许多电子阅读器的支持，并且大多数智能手机、平板电脑和计算机上都可以使用兼容的软件。

这部分介绍如何将 `.epub` 文档加载到我们可以在后续使用的文档格式中。您需要安装 [`pandoc`](https://pandoc.org/installing.html) 包，以使此加载器正常工作，例如在 OSX 上使用 `brew install pandoc`。

请参阅 [此指南](/docs/integrations/providers/unstructured/) 获取有关本地设置 Unstructured 的更多说明，包括设置所需的系统依赖项。


```python
%pip install --upgrade --quiet unstructured
```


```python
from langchain_community.document_loaders import UnstructuredEPubLoader

loader = UnstructuredEPubLoader("./example_data/childrens-literature.epub")

data = loader.load()

data[0]
```



```output
```

## 保留元素

在后台，Unstructured 为不同的文本块创建不同的“元素”。默认情况下，我们将这些元素组合在一起，但您可以通过指定 `mode="elements"` 来轻松保持这种分离。

```python
loader = UnstructuredEPubLoader(
    "./example_data/childrens-literature.epub", mode="elements"
)

data = loader.load()

data[0]
```

```output
Document(page_content='Guide', metadata={'source': './example_data/childrens-literature.epub', 'category_depth': 1, 'last_modified': '2024-07-01T11:12:08', 'languages': ['eng'], 'filetype': 'application/epub', 'file_directory': './example_data', 'filename': 'childrens-literature.epub', 'category': 'Title'})
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)