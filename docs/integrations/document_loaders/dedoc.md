---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/dedoc.ipynb
---

# Dedoc

此示例演示了将 `Dedoc` 与 `LangChain` 结合使用作为 `DocumentLoader` 的方法。

## 概述

[Dedoc](https://dedoc.readthedocs.io) 是一个 [开源](https://github.com/ispras/dedoc) 库/服务，能够从各种格式的文件中提取文本、表格、附加文件和文档结构（例如，标题、列表项等）。

`Dedoc` 支持 `DOCX`、`XLSX`、`PPTX`、`EML`、`HTML`、`PDF`、图像等。支持的格式完整列表可以在 [这里](https://dedoc.readthedocs.io/en/latest/#id1) 找到。

### 集成细节

| 类别                                                                                                                                               | 包                                                                                             | 本地 | 可序列化 | JS 支持 |
|:--------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------|:-----:|:--------:|:-------:|
| [DedocFileLoader](https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.dedoc.DedocFileLoader.html)       | [langchain_community](https://api.python.langchain.com/en/latest/community_api_reference.html) |   ❌   |     beta |     ❌   |
| [DedocPDFLoader](https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.pdf.DedocPDFLoader.html)           | [langchain_community](https://api.python.langchain.com/en/latest/community_api_reference.html) |   ❌   |     beta |     ❌   | 
| [DedocAPIFileLoader](https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.dedoc.DedocAPIFileLoader.html) | [langchain_community](https://api.python.langchain.com/en/latest/community_api_reference.html) |   ❌   |     beta |     ❌   |

### 加载器特性

懒加载和异步加载的方法是可用的，但实际上，文档加载是同步执行的。

|       来源        | 文档懒加载 | 异步支持 |
|:-----------------:|:----------:|:--------:| 
|  DedocFileLoader  |     ❌     |    ❌    |
|   DedocPDFLoader  |     ❌     |    ❌    | 
| DedocAPIFileLoader|     ❌     |    ❌    |

## 设置

* 要访问 `DedocFileLoader` 和 `DedocPDFLoader` 文档加载器，您需要安装 `dedoc` 集成包。
* 要访问 `DedocAPIFileLoader`，您需要运行 `Dedoc` 服务，例如 `Docker` 容器（有关更多详细信息，请参见 [文档](https://dedoc.readthedocs.io/en/latest/getting_started/installation.html#install-and-run-dedoc-using-docker)）：

```bash
docker pull dedocproject/dedoc
docker run -p 1231:1231
```

`Dedoc` 安装说明在 [这里](https://dedoc.readthedocs.io/en/latest/getting_started/installation.html)。

```python
# 安装包
%pip install --quiet "dedoc[torch]"
```
```output
注意：您可能需要重启内核以使用更新的包。
```

## 实例化


```python
from langchain_community.document_loaders import DedocFileLoader

loader = DedocFileLoader("./example_data/state_of_the_union.txt")
```

## 加载


```python
docs = loader.load()
docs[0].page_content[:100]
```



```output
'\nMadam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and t'
```

## 懒加载


```python
docs = loader.lazy_load()

for doc in docs:
    print(doc.page_content[:100])
    break
```
```output

Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and t
```

## API 参考

有关配置和调用 `Dedoc` 加载器的详细信息，请参阅 API 参考：

* https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.dedoc.DedocFileLoader.html
* https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.pdf.DedocPDFLoader.html
* https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.dedoc.DedocAPIFileLoader.html

## 加载任何文件

为了自动处理任何[支持的格式](https://dedoc.readthedocs.io/en/latest/#id1)的文件，`DedocFileLoader` 可以派上用场。文件加载器会自动检测文件类型及其正确的扩展名。

文件解析过程可以通过在 `DedocFileLoader` 类初始化时使用 `dedoc_kwargs` 进行配置。这里给出了某些选项使用的基本示例，更多关于配置参数的详细信息，请参阅 `DedocFileLoader` 的文档和 [dedoc 文档](https://dedoc.readthedocs.io/en/latest/parameters/parameters.html)。

### 基本示例


```python
from langchain_community.document_loaders import DedocFileLoader

loader = DedocFileLoader("./example_data/state_of_the_union.txt")

docs = loader.load()

docs[0].page_content[:400]
```



```output
'\nMadam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.  \n\n\n\nLast year COVID-19 kept us apart. This year we are finally together again. \n\n\n\nTonight, we meet as Democrats Republicans and Independents. But most importantly as Americans. \n\n\n\nWith a duty to one another to the American people to '
```

### 切分模式

`DedocFileLoader` 支持将文档分割成多个部分（每个部分单独返回）。为此，使用 `split` 参数，选项如下：
* `document`（默认值）：文档文本作为单个 langchain `Document` 对象返回（不进行切分）；
* `page`：将文档文本按页切分（适用于 `PDF`、`DJVU`、`PPTX`、`PPT`、`ODP`）；
* `node`：将文档文本切分为 `Dedoc` 树节点（标题节点、列表项节点、原始文本节点）；
* `line`：将文档文本按行切分。


```python
loader = DedocFileLoader(
    "./example_data/layout-parser-paper.pdf",
    split="page",
    pages=":2",
)

docs = loader.load()

len(docs)
```


```output
2
```

### 处理表格

`DedocFileLoader` 在加载器初始化时将 `with_tables` 参数设置为 `True` 时支持表格处理（默认 `with_tables=True`）。

表格不会被拆分 - 每个表格对应一个 langchain `Document` 对象。
对于表格，`Document` 对象具有额外的 `metadata` 字段 `type="table"` 和 `text_as_html`，其包含表格的 `HTML` 表示形式。


```python
loader = DedocFileLoader("./example_data/mlb_teams_2012.csv")

docs = loader.load()

docs[1].metadata["type"], docs[1].metadata["text_as_html"][:200]
```



```output
('table',
 '<table border="1" style="border-collapse: collapse; width: 100%;">\n<tbody>\n<tr>\n<td colspan="1" rowspan="1">Team</td>\n<td colspan="1" rowspan="1"> &quot;Payroll (millions)&quot;</td>\n<td colspan="1" r')
```

### 处理附件文件

`DedocFileLoader` 在加载器初始化时将 `with_attachments` 设置为 `True` 时支持附件文件处理（默认情况下为 `with_attachments=False`）。

附件根据 `split` 参数进行拆分。
对于附件，langchain 的 `Document` 对象具有额外的元数据字段 `type="attachment"`。


```python
loader = DedocFileLoader(
    "./example_data/fake-email-attachment.eml",
    with_attachments=True,
)

docs = loader.load()

docs[1].metadata["type"], docs[1].page_content
```



```output
('attachment',
 '\nContent-Type\nmultipart/mixed; boundary="0000000000005d654405f082adb7"\nDate\nFri, 23 Dec 2022 12:08:48 -0600\nFrom\nMallori Harrell <mallori@unstructured.io>\nMIME-Version\n1.0\nMessage-ID\n<CAPgNNXSzLVJ-d1OCX_TjFgJU7ugtQrjFybPtAMmmYZzphxNFYg@mail.gmail.com>\nSubject\nFake email with attachment\nTo\nMallori Harrell <mallori@unstructured.io>')
```

## 加载 PDF 文件

如果您只想处理 `PDF` 文档，可以使用仅支持 `PDF` 的 `DedocPDFLoader`。加载器支持相同的参数用于文档拆分、表格和附件提取。

`Dedoc` 可以提取带有或不带有文本层的 `PDF`，并能够自动检测其存在性和正确性。提供了几种 `PDF` 处理程序，您可以使用 `pdf_with_text_layer` 参数选择其中之一。有关更多详细信息，请参见 [参数说明](https://dedoc.readthedocs.io/en/latest/parameters/pdf_handling.html)。

对于没有文本层的 `PDF`，需要安装 `Tesseract OCR` 及其语言包。在这种情况下，[该说明](https://dedoc.readthedocs.io/en/latest/tutorials/add_new_language.html) 可能会有所帮助。

```python
from langchain_community.document_loaders import DedocPDFLoader

loader = DedocPDFLoader(
    "./example_data/layout-parser-paper.pdf", pdf_with_text_layer="true", pages="2:2"
)

docs = loader.load()

docs[0].page_content[:400]
```

```output
'\n2\n\nZ. Shen et al.\n\n37], layout detection [38, 22], table detection [26], and scene text detection [4].\n\nA generalized learning-based framework dramatically reduces the need for the\n\nmanual speciﬁcation of complicated rules, which is the status quo with traditional\n\nmethods. DL has the potential to transform DIA pipelines and beneﬁt a broad\n\nspectrum of large-scale document digitization projects.\n'
```

## Dedoc API

如果您想要快速启动并减少设置，可以将 `Dedoc` 作为服务使用。  
**`DedocAPIFileLoader` 可以在不安装 `dedoc` 库的情况下使用。**  
该加载器支持与 `DedocFileLoader` 相同的参数，并且还会自动检测输入文件类型。

要使用 `DedocAPIFileLoader`，您需要运行 `Dedoc` 服务，例如 `Docker` 容器（请参阅 [文档](https://dedoc.readthedocs.io/en/latest/getting_started/installation.html#install-and-run-dedoc-using-docker) 以获取更多详细信息）：

```bash
docker pull dedocproject/dedoc
docker run -p 1231:1231
```

请不要在您的代码中使用我们的演示 URL `https://dedoc-readme.hf.space`。

```python
from langchain_community.document_loaders import DedocAPIFileLoader

loader = DedocAPIFileLoader(
    "./example_data/state_of_the_union.txt",
    url="https://dedoc-readme.hf.space",
)

docs = loader.load()

docs[0].page_content[:400]
```

```output
'\nMadam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and the Cabinet. Justices of the Supreme Court. My fellow Americans.  \n\n\n\nLast year COVID-19 kept us apart. This year we are finally together again. \n\n\n\nTonight, we meet as Democrats Republicans and Independents. But most importantly as Americans. \n\n\n\nWith a duty to one another to the American people to '
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)