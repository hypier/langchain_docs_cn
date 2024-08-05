---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/document_loader_custom.ipynb
title: 自定义文档加载器
sidebar_position: 10
---

# 如何创建自定义文档加载器

## 概述

基于 LLM 的应用程序通常涉及从数据库或文件（如 PDF）中提取数据，并将其转换为 LLM 可以使用的格式。在 LangChain 中，这通常涉及创建 Document 对象，该对象封装了提取的文本（`page_content`）以及元数据——一个包含有关文档的详细信息的字典，例如作者的姓名或出版日期。

`Document` 对象通常格式化为提示，输入到 LLM 中，使 LLM 能够使用 `Document` 中的信息生成所需的响应（例如，总结文档）。`Documents` 可以立即使用，也可以索引到向量存储中以便将来检索和使用。

文档加载的主要抽象是：

| 组件           | 描述                           |
|----------------|--------------------------------|
| Document       | 包含 `text` 和 `metadata`     |
| BaseLoader     | 用于将原始数据转换为 `Documents`  |
| Blob           | 表示位于文件或内存中的二进制数据 |
| BaseBlobParser | 解析 `Blob` 以生成 `Document` 对象的逻辑 |

本指南将演示如何编写自定义文档加载和文件解析逻辑；具体而言，我们将看到如何：

1. 通过从 `BaseLoader` 子类化创建标准文档加载器。
2. 使用 `BaseBlobParser` 创建解析器，并将其与 `Blob` 和 `BlobLoaders` 一起使用。这在处理文件时特别有用。

## 标准文档加载器

文档加载器可以通过从 `BaseLoader` 子类化来实现，该接口提供了加载文档的标准接口。

### 接口 

| 方法名称    | 说明        |
|-------------|-------------|
| lazy_load   | 用于**懒惰**地逐个加载文档。用于生产代码。 |
| alazy_load  | `lazy_load` 的异步变体 |
| load        | 用于**急切**地将所有文档加载到内存中。用于原型设计或交互式工作。 |
| aload       | 用于**急切**地将所有文档加载到内存中。用于原型设计或交互式工作。 **于2024-04新增至LangChain。** |

* `load` 方法是一个便利方法，仅用于原型设计工作 -- 它只是调用 `list(self.lazy_load())`。
* `alazy_load` 有一个默认实现，将委派给 `lazy_load`。如果您使用异步，我们建议覆盖默认实现并提供原生异步实现。

::: {.callout-important}
在实现文档加载器时，**请不要**通过 `lazy_load` 或 `alazy_load` 方法提供参数。

所有配置应通过初始化器 (__init__) 传递。这是LangChain做出的设计选择，以确保一旦实例化了文档加载器，就拥有加载文档所需的所有信息。
:::

### 实现

让我们创建一个标准文档加载器的示例，该加载器加载文件并从文件中的每一行创建文档。

```python
from typing import AsyncIterator, Iterator

from langchain_core.document_loaders import BaseLoader
from langchain_core.documents import Document


class CustomDocumentLoader(BaseLoader):
    """一个示例文档加载器，逐行读取文件。"""

    def __init__(self, file_path: str) -> None:
        """使用文件路径初始化加载器。

        Args:
            file_path: 要加载的文件的路径。
        """
        self.file_path = file_path

    def lazy_load(self) -> Iterator[Document]:  # <-- 不接受任何参数
        """一个懒加载器，逐行读取文件。

        当你实现懒加载方法时，应该使用生成器
        一次生成一个文档。
        """
        with open(self.file_path, encoding="utf-8") as f:
            line_number = 0
            for line in f:
                yield Document(
                    page_content=line,
                    metadata={"line_number": line_number, "source": self.file_path},
                )
                line_number += 1

    # alazy_load 是可选的。
    # 如果你省略实现，将使用一个默认实现，该实现委托给 lazy_load！
    async def alazy_load(
        self,
    ) -> AsyncIterator[Document]:  # <-- 不接受任何参数
        """一个异步懒加载器，逐行读取文件。"""
        # 需要 aiofiles
        # 使用 `pip install aiofiles` 安装
        # https://github.com/Tinche/aiofiles
        import aiofiles

        async with aiofiles.open(self.file_path, encoding="utf-8") as f:
            line_number = 0
            async for line in f:
                yield Document(
                    page_content=line,
                    metadata={"line_number": line_number, "source": self.file_path},
                )
                line_number += 1
```

### 测试 🧪

为了测试文档加载器，我们需要一个包含优质内容的文件。

```python
with open("./meow.txt", "w", encoding="utf-8") as f:
    quality_content = "meow meow🐱 \n meow meow🐱 \n meow😻😻"
    f.write(quality_content)

loader = CustomDocumentLoader("./meow.txt")
```

```python
## 测试懒加载接口
for doc in loader.lazy_load():
    print()
    print(type(doc))
    print(doc)
```
```output

<class 'langchain_core.documents.base.Document'>
page_content='meow meow🐱 \n' metadata={'line_number': 0, 'source': './meow.txt'}

<class 'langchain_core.documents.base.Document'>
page_content=' meow meow🐱 \n' metadata={'line_number': 1, 'source': './meow.txt'}

<class 'langchain_core.documents.base.Document'>
page_content=' meow😻😻' metadata={'line_number': 2, 'source': './meow.txt'}
```

```python
## 测试异步实现
async for doc in loader.alazy_load():
    print()
    print(type(doc))
    print(doc)
```
```output

<class 'langchain_core.documents.base.Document'>
page_content='meow meow🐱 \n' metadata={'line_number': 0, 'source': './meow.txt'}

<class 'langchain_core.documents.base.Document'>
page_content=' meow meow🐱 \n' metadata={'line_number': 1, 'source': './meow.txt'}

<class 'langchain_core.documents.base.Document'>
page_content=' meow😻😻' metadata={'line_number': 2, 'source': './meow.txt'}
```
::: {.callout-tip}

`load()` 在交互环境中，例如 jupyter notebook，可以非常有用。

避免在生产代码中使用它，因为急切加载假设所有内容都可以适应内存，这并不总是成立，尤其是对于企业数据。
:::

```python
loader.load()
```

```output
[Document(page_content='meow meow🐱 \n', metadata={'line_number': 0, 'source': './meow.txt'}),
 Document(page_content=' meow meow🐱 \n', metadata={'line_number': 1, 'source': './meow.txt'}),
 Document(page_content=' meow😻😻', metadata={'line_number': 2, 'source': './meow.txt'})]
```

## 文件处理

许多文档加载器涉及解析文件。这些加载器之间的区别通常源于文件的解析方式，而不是文件的加载方式。例如，您可以使用 `open` 来读取 PDF 或 markdown 文件的二进制内容，但您需要不同的解析逻辑将该二进制数据转换为文本。

因此，将解析逻辑与加载逻辑解耦是有帮助的，这使得无论数据是如何加载的，都更容易重用给定的解析器。

### BaseBlobParser

`BaseBlobParser` 是一个接口，它接受一个 `blob` 并输出一个 `Document` 对象的列表。`blob` 是一种数据的表示形式，可以存储在内存中或文件中。LangChain python 有一个 `Blob` 原语，灵感来自于 [Blob WebAPI 规范](https://developer.mozilla.org/en-US/docs/Web/API/Blob)。

```python
from langchain_core.document_loaders import BaseBlobParser, Blob


class MyParser(BaseBlobParser):
    """一个简单的解析器，从每一行创建一个文档。"""

    def lazy_parse(self, blob: Blob) -> Iterator[Document]:
        """逐行解析 blob 为文档。"""
        line_number = 0
        with blob.as_bytes_io() as f:
            for line in f:
                line_number += 1
                yield Document(
                    page_content=line,
                    metadata={"line_number": line_number, "source": blob.source},
                )
```


```python
blob = Blob.from_path("./meow.txt")
parser = MyParser()
```


```python
list(parser.lazy_parse(blob))
```



```output
[Document(page_content='meow meow🐱 \n', metadata={'line_number': 1, 'source': './meow.txt'}),
 Document(page_content=' meow meow🐱 \n', metadata={'line_number': 2, 'source': './meow.txt'}),
 Document(page_content=' meow😻😻', metadata={'line_number': 3, 'source': './meow.txt'})]
```


使用 **blob** API 还可以直接从内存加载内容，而无需从文件中读取！


```python
blob = Blob(data=b"some data from memory\nmeow")
list(parser.lazy_parse(blob))
```



```output
[Document(page_content='some data from memory\n', metadata={'line_number': 1, 'source': None}),
 Document(page_content='meow', metadata={'line_number': 2, 'source': None})]
```

### Blob

让我们快速浏览一下 Blob API。


```python
blob = Blob.from_path("./meow.txt", metadata={"foo": "bar"})
```


```python
blob.encoding
```



```output
'utf-8'
```



```python
blob.as_bytes()
```



```output
b'meow meow\xf0\x9f\x90\xb1 \n meow meow\xf0\x9f\x90\xb1 \n meow\xf0\x9f\x98\xbb\xf0\x9f\x98\xbb'
```



```python
blob.as_string()
```



```output
'meow meow🐱 \n meow meow🐱 \n meow😻😻'
```



```python
blob.as_bytes_io()
```



```output
<contextlib._GeneratorContextManager at 0x743f34324450>
```



```python
blob.metadata
```



```output
{'foo': 'bar'}
```



```python
blob.source
```



```output
'./meow.txt'
```

### Blob Loaders

虽然解析器封装了解析二进制数据为文档所需的逻辑，但 *blob loaders* 封装了从给定存储位置加载 blobs 所需的逻辑。

目前，`LangChain` 仅支持 `FileSystemBlobLoader`。

您可以使用 `FileSystemBlobLoader` 来加载 blobs，然后使用解析器对其进行解析。

```python
from langchain_community.document_loaders.blob_loaders import FileSystemBlobLoader

blob_loader = FileSystemBlobLoader(path=".", glob="*.mdx", show_progress=True)
```

```python
parser = MyParser()
for blob in blob_loader.yield_blobs():
    for doc in parser.lazy_parse(blob):
        print(doc)
        break
```

```output
  0%|          | 0/8 [00:00<?, ?it/s]
```
```output
page_content='# Microsoft Office\n' metadata={'line_number': 1, 'source': 'office_file.mdx'}
page_content='# Markdown\n' metadata={'line_number': 1, 'source': 'markdown.mdx'}
page_content='# JSON\n' metadata={'line_number': 1, 'source': 'json.mdx'}
page_content='---\n' metadata={'line_number': 1, 'source': 'pdf.mdx'}
page_content='---\n' metadata={'line_number': 1, 'source': 'index.mdx'}
page_content='# File Directory\n' metadata={'line_number': 1, 'source': 'file_directory.mdx'}
page_content='# CSV\n' metadata={'line_number': 1, 'source': 'csv.mdx'}
page_content='# HTML\n' metadata={'line_number': 1, 'source': 'html.mdx'}
```

### 通用加载器

LangChain 有一个 `GenericLoader` 抽象，它将 `BlobLoader` 与 `BaseBlobParser` 组合在一起。

`GenericLoader` 旨在提供标准化的类方法，使得使用现有的 `BlobLoader` 实现变得简单。目前，仅支持 `FileSystemBlobLoader`。

```python
from langchain_community.document_loaders.generic import GenericLoader

loader = GenericLoader.from_filesystem(
    path=".", glob="*.mdx", show_progress=True, parser=MyParser()
)

for idx, doc in enumerate(loader.lazy_load()):
    if idx < 5:
        print(doc)

print("... output truncated for demo purposes")
```

```output
  0%|          | 0/8 [00:00<?, ?it/s]
```
```output
page_content='# Microsoft Office\n' metadata={'line_number': 1, 'source': 'office_file.mdx'}
page_content='\n' metadata={'line_number': 2, 'source': 'office_file.mdx'}
page_content='>[The Microsoft Office](https://www.office.com/) suite of productivity software includes Microsoft Word, Microsoft Excel, Microsoft PowerPoint, Microsoft Outlook, and Microsoft OneNote. It is available for Microsoft Windows and macOS operating systems. It is also available on Android and iOS.\n' metadata={'line_number': 3, 'source': 'office_file.mdx'}
page_content='\n' metadata={'line_number': 4, 'source': 'office_file.mdx'}
page_content='This covers how to load commonly used file formats including `DOCX`, `XLSX` and `PPTX` documents into a document format that we can use downstream.\n' metadata={'line_number': 5, 'source': 'office_file.mdx'}
... output truncated for demo purposes
```
#### 自定义通用加载器

如果你真的喜欢创建类，你可以子类化并创建一个类来封装逻辑。

你可以从这个类中子类化，以使用现有的加载器加载内容。

```python
from typing import Any


class MyCustomLoader(GenericLoader):
    @staticmethod
    def get_parser(**kwargs: Any) -> BaseBlobParser:
        """重写此方法以将默认解析器与类关联起来。"""
        return MyParser()
```

```python
loader = MyCustomLoader.from_filesystem(path=".", glob="*.mdx", show_progress=True)

for idx, doc in enumerate(loader.lazy_load()):
    if idx < 5:
        print(doc)

print("... output truncated for demo purposes")
```

```output
  0%|          | 0/8 [00:00<?, ?it/s]
```
```output
page_content='# Microsoft Office\n' metadata={'line_number': 1, 'source': 'office_file.mdx'}
page_content='\n' metadata={'line_number': 2, 'source': 'office_file.mdx'}
page_content='>[The Microsoft Office](https://www.office.com/) suite of productivity software includes Microsoft Word, Microsoft Excel, Microsoft PowerPoint, Microsoft Outlook, and Microsoft OneNote. It is available for Microsoft Windows and macOS operating systems. It is also available on Android and iOS.\n' metadata={'line_number': 3, 'source': 'office_file.mdx'}
page_content='\n' metadata={'line_number': 4, 'source': 'office_file.mdx'}
page_content='This covers how to load commonly used file formats including `DOCX`, `XLSX` and `PPTX` documents into a document format that we can use downstream.\n' metadata={'line_number': 5, 'source': 'office_file.mdx'}
... output truncated for demo purposes
```