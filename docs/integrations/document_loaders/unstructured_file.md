---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/unstructured_file.ipynb
---

# Unstructured

本笔记本介绍如何使用 `Unstructured` 包加载多种类型的文件。`Unstructured` 目前支持加载文本文件、PowerPoint、HTML、PDF、图像等。

请参阅 [此指南](/docs/integrations/providers/unstructured/) 获取有关本地设置 Unstructured 的更多说明，包括设置所需的系统依赖项。

```python
# Install package, compatible with API partitioning
%pip install --upgrade --quiet "langchain-unstructured"
```

### 本地分区（可选）

默认情况下，`langchain-unstructured` 安装了一个较小的版本，需要将分区逻辑卸载到 Unstructured API，这需要一个 `api_key`。有关使用 API 进行分区的详细信息，请参阅下面的 Unstructured API 部分。

如果您希望在本地运行分区逻辑，您需要安装一组合系统依赖项，如 
[Unstructured 文档中所述](https://docs.unstructured.io/open-source/installation/full-installation)。

例如，在 Mac 上，您可以通过以下命令安装所需的依赖项：

```bash
# base dependencies
brew install libmagic poppler tesseract

# If parsing xml / html documents:
brew install libxml2 libxslt
```

您可以通过以下命令安装所需的 `pip` 依赖项：

```bash
pip install "langchain-unstructured[local]"
```

### 快速开始

要简单地将文件加载为文档，您可以使用 LangChain `DocumentLoader.load` 
接口：

```python
from langchain_unstructured import UnstructuredLoader

loader = UnstructuredLoader("./example_data/state_of_the_union.txt")

docs = loader.load()
```

### 加载文件列表


```python
file_paths = [
    "./example_data/whatsapp_chat.txt",
    "./example_data/state_of_the_union.txt",
]

loader = UnstructuredLoader(file_paths)

docs = loader.load()

print(docs[0].metadata.get("filename"), ": ", docs[0].page_content[:100])
print(docs[-1].metadata.get("filename"), ": ", docs[-1].page_content[:100])
```
```output
whatsapp_chat.txt :  1/22/23, 6:30 PM - User 1: Hi! Im interested in your bag. Im offering $50. Let me know if you are in
state_of_the_union.txt :  May God bless you all. May God protect our troops.
```

## PDF 示例

处理 PDF 文档的方式完全相同。Unstructured 检测文件类型并提取相同类型的元素。

### 定义分区策略

Unstructured文档加载器允许用户传入`strategy`参数，告诉Unstructured如何对pdf和其他OCR文档进行分区。目前支持的策略有`"auto"`、`"hi_res"`、`"ocr_only"`和`"fast"`。了解更多不同策略的信息，请访问[这里](https://docs.unstructured.io/open-source/core-functionality/partitioning#partition-pdf)。

并非所有文档类型都有单独的高分辨率和快速分区策略。对于这些文档类型，`strategy`关键字参数将被忽略。在某些情况下，如果缺少依赖项（即文档分区模型），高分辨率策略将回退为快速策略。您可以在下面查看如何将策略应用于`UnstructuredLoader`。

```python
from langchain_unstructured import UnstructuredLoader

loader = UnstructuredLoader("./example_data/layout-parser-paper.pdf", strategy="fast")

docs = loader.load()

docs[5:10]
```

```output
[Document(metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((16.34, 393.9), (16.34, 560.0), (36.34, 560.0), (36.34, 393.9)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'file_directory': './example_data', 'filename': 'layout-parser-paper.pdf', 'languages': ['eng'], 'last_modified': '2024-02-27T15:49:27', 'page_number': 1, 'parent_id': '89565df026a24279aaea20dc08cedbec', 'filetype': 'application/pdf', 'category': 'UncategorizedText', 'element_id': 'e9fa370aef7ee5c05744eb7bb7d9981b'}, page_content='2 v 8 4 3 5 1 . 3 0 1 2 : v i X r a'),
 Document(metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((157.62199999999999, 114.23496279999995), (157.62199999999999, 146.5141628), (457.7358962799999, 146.5141628), (457.7358962799999, 114.23496279999995)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'file_directory': './example_data', 'filename': 'layout-parser-paper.pdf', 'languages': ['eng'], 'last_modified': '2024-02-27T15:49:27', 'page_number': 1, 'filetype': 'application/pdf', 'category': 'Title', 'element_id': 'bde0b230a1aa488e3ce837d33015181b'}, page_content='LayoutParser: A Uniﬁed Toolkit for Deep Learning Based Document Image Analysis'),
 Document(metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((134.809, 168.64029940800003), (134.809, 192.2517444), (480.5464199080001, 192.2517444), (480.5464199080001, 168.64029940800003)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'file_directory': './example_data', 'filename': 'layout-parser-paper.pdf', 'languages': ['eng'], 'last_modified': '2024-02-27T15:49:27', 'page_number': 1, 'parent_id': 'bde0b230a1aa488e3ce837d33015181b', 'filetype': 'application/pdf', 'category': 'UncategorizedText', 'element_id': '54700f902899f0c8c90488fa8d825bce'}, page_content='Zejiang Shen1 ((cid:0)), Ruochen Zhang2, Melissa Dell3, Benjamin Charles Germain Lee4, Jacob Carlson3, and Weining Li5'),
 Document(metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((207.23000000000002, 202.57205439999996), (207.23000000000002, 311.8195408), (408.12676, 311.8195408), (408.12676, 202.57205439999996)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'file_directory': './example_data', 'filename': 'layout-parser-paper.pdf', 'languages': ['eng'], 'last_modified': '2024-02-27T15:49:27', 'page_number': 1, 'parent_id': 'bde0b230a1aa488e3ce837d33015181b', 'filetype': 'application/pdf', 'category': 'UncategorizedText', 'element_id': 'b650f5867bad9bb4e30384282c79bcfe'}, page_content='1 Allen Institute for AI shannons@allenai.org 2 Brown University ruochen zhang@brown.edu 3 Harvard University {melissadell,jacob carlson}@fas.harvard.edu 4 University of Washington bcgl@cs.washington.edu 5 University of Waterloo w422li@uwaterloo.ca'),
 Document(metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((162.779, 338.45008160000003), (162.779, 566.8455408), (454.0372021523199, 566.8455408), (454.0372021523199, 338.45008160000003)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'file_directory': './example_data', 'filename': 'layout-parser-paper.pdf', 'languages': ['eng'], 'last_modified': '2024-02-27T15:49:27', 'links': [{'text': ':// layout - parser . github . io', 'url': 'https://layout-parser.github.io', 'start_index': 1477}], 'page_number': 1, 'parent_id': 'bde0b230a1aa488e3ce837d33015181b', 'filetype': 'application/pdf', 'category': 'NarrativeText', 'element_id': 'cfc957c94fe63c8fd7c7f4bcb56e75a7'}, page_content='Abstract. Recent advances in document image analysis (DIA) have been primarily driven by the application of neural networks. Ideally, research outcomes could be easily deployed in production and extended for further investigation. However, various factors like loosely organized codebases and sophisticated model conﬁgurations complicate the easy reuse of im- portant innovations by a wide audience. Though there have been on-going eﬀorts to improve reusability and simplify deep learning (DL) model development in disciplines like natural language processing and computer vision, none of them are optimized for challenges in the domain of DIA. This represents a major gap in the existing toolkit, as DIA is central to academic research across a wide range of disciplines in the social sciences and humanities. This paper introduces LayoutParser, an open-source library for streamlining the usage of DL in DIA research and applica- tions. The core LayoutParser library comes with a set of simple and intuitive interfaces for applying and customizing DL models for layout de- tection, character recognition, and many other document processing tasks. To promote extensibility, LayoutParser also incorporates a community platform for sharing both pre-trained models and full document digiti- zation pipelines. We demonstrate that LayoutParser is helpful for both lightweight and large-scale digitization pipelines in real-word use cases. The library is publicly available at https://layout-parser.github.io.')]
```

## 后处理

如果您需要在提取后对 `unstructured` 元素进行后处理，可以在实例化 `UnstructuredLoader` 时将 `str` -> `str` 函数的列表传递给 `post_processors` 参数。这同样适用于其他 Unstructured 加载器。以下是一个示例。

```python
from langchain_unstructured import UnstructuredLoader
from unstructured.cleaners.core import clean_extra_whitespace

loader = UnstructuredLoader(
    "./example_data/layout-parser-paper.pdf",
    post_processors=[clean_extra_whitespace],
)

docs = loader.load()

docs[5:10]
```

```output
[Document(metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((16.34, 393.9), (16.34, 560.0), (36.34, 560.0), (36.34, 393.9)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'file_directory': './example_data', 'filename': 'layout-parser-paper.pdf', 'languages': ['eng'], 'last_modified': '2024-02-27T15:49:27', 'page_number': 1, 'parent_id': '89565df026a24279aaea20dc08cedbec', 'filetype': 'application/pdf', 'category': 'UncategorizedText', 'element_id': 'e9fa370aef7ee5c05744eb7bb7d9981b'}, page_content='2 v 8 4 3 5 1 . 3 0 1 2 : v i X r a'),
 Document(metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((157.62199999999999, 114.23496279999995), (157.62199999999999, 146.5141628), (457.7358962799999, 146.5141628), (457.7358962799999, 114.23496279999995)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'file_directory': './example_data', 'filename': 'layout-parser-paper.pdf', 'languages': ['eng'], 'last_modified': '2024-02-27T15:49:27', 'page_number': 1, 'filetype': 'application/pdf', 'category': 'Title', 'element_id': 'bde0b230a1aa488e3ce837d33015181b'}, page_content='LayoutParser: A Uniﬁed Toolkit for Deep Learning Based Document Image Analysis'),
 Document(metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((134.809, 168.64029940800003), (134.809, 192.2517444), (480.5464199080001, 192.2517444), (480.5464199080001, 168.64029940800003)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'file_directory': './example_data', 'filename': 'layout-parser-paper.pdf', 'languages': ['eng'], 'last_modified': '2024-02-27T15:49:27', 'page_number': 1, 'parent_id': 'bde0b230a1aa488e3ce837d33015181b', 'filetype': 'application/pdf', 'category': 'UncategorizedText', 'element_id': '54700f902899f0c8c90488fa8d825bce'}, page_content='Zejiang Shen1 ((cid:0)), Ruochen Zhang2, Melissa Dell3, Benjamin Charles Germain Lee4, Jacob Carlson3, and Weining Li5'),
 Document(metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((207.23000000000002, 202.57205439999996), (207.23000000000002, 311.8195408), (408.12676, 311.8195408), (408.12676, 202.57205439999996)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'file_directory': './example_data', 'filename': 'layout-parser-paper.pdf', 'languages': ['eng'], 'last_modified': '2024-02-27T15:49:27', 'page_number': 1, 'parent_id': 'bde0b230a1aa488e3ce837d33015181b', 'filetype': 'application/pdf', 'category': 'UncategorizedText', 'element_id': 'b650f5867bad9bb4e30384282c79bcfe'}, page_content='1 Allen Institute for AI shannons@allenai.org 2 Brown University ruochen zhang@brown.edu 3 Harvard University {melissadell,jacob carlson}@fas.harvard.edu 4 University of Washington bcgl@cs.washington.edu 5 University of Waterloo w422li@uwaterloo.ca'),
 Document(metadata={'source': './example_data/layout-parser-paper.pdf', 'coordinates': {'points': ((162.779, 338.45008160000003), (162.779, 566.8455408), (454.0372021523199, 566.8455408), (454.0372021523199, 338.45008160000003)), 'system': 'PixelSpace', 'layout_width': 612, 'layout_height': 792}, 'file_directory': './example_data', 'filename': 'layout-parser-paper.pdf', 'languages': ['eng'], 'last_modified': '2024-02-27T15:49:27', 'links': [{'text': ':// layout - parser . github . io', 'url': 'https://layout-parser.github.io', 'start_index': 1477}], 'page_number': 1, 'parent_id': 'bde0b230a1aa488e3ce837d33015181b', 'filetype': 'application/pdf', 'category': 'NarrativeText', 'element_id': 'cfc957c94fe63c8fd7c7f4bcb56e75a7'}, page_content='Abstract. Recent advances in document image analysis (DIA) have been primarily driven by the application of neural networks. Ideally, research outcomes could be easily deployed in production and extended for further investigation. However, various factors like loosely organized codebases and sophisticated model conﬁgurations complicate the easy reuse of im- portant innovations by a wide audience. Though there have been on-going eﬀorts to improve reusability and simplify deep learning (DL) model development in disciplines like natural language processing and computer vision, none of them are optimized for challenges in the domain of DIA. This represents a major gap in the existing toolkit, as DIA is central to academic research across a wide range of disciplines in the social sciences and humanities. This paper introduces LayoutParser, an open-source library for streamlining the usage of DL in DIA research and applica- tions. The core LayoutParser library comes with a set of simple and intuitive interfaces for applying and customizing DL models for layout de- tection, character recognition, and many other document processing tasks. To promote extensibility, LayoutParser also incorporates a community platform for sharing both pre-trained models and full document digiti- zation pipelines. We demonstrate that LayoutParser is helpful for both lightweight and large-scale digitization pipelines in real-word use cases. The library is publicly available at https://layout-parser.github.io.')]
```

## Unstructured API

如果您想快速开始使用较小的包并获取最新的分区信息，可以使用 `pip install unstructured-client` 和 `pip install langchain-unstructured`。有关 `UnstructuredLoader` 的更多信息，请参考 [Unstructured provider page](https://python.langchain.com/v0.1/docs/integrations/document_loaders/unstructured_file/)。

当您传入 `api_key` 并设置 `partition_via_api=True` 时，加载器将使用托管的 Unstructured 无服务器 API 处理您的文档。您可以在 [这里](https://unstructured.io/api-key/) 生成免费的 Unstructured API 密钥。

如果您想自托管 Unstructured API 或在本地运行，请查看 [这里](https://github.com/Unstructured-IO/unstructured-api#dizzy-instructions-for-using-the-docker-image) 的说明。


```python
# Install package
%pip install "langchain-unstructured"
%pip install "unstructured-client"

# Set API key
import os

os.environ["UNSTRUCTURED_API_KEY"] = "FAKE_API_KEY"
```


```python
from langchain_unstructured import UnstructuredLoader

loader = UnstructuredLoader(
    file_path="example_data/fake.docx",
    api_key=os.getenv("UNSTRUCTURED_API_KEY"),
    partition_via_api=True,
)

docs = loader.load()
docs[0]
```
```output
INFO: Preparing to split document for partition.
INFO: Given file doesn't have '.pdf' extension, so splitting is not enabled.
INFO: Partitioning without split.
INFO: Successfully partitioned the document.
```


```output
Document(metadata={'source': 'example_data/fake.docx', 'category_depth': 0, 'filename': 'fake.docx', 'languages': ['por', 'cat'], 'filetype': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document', 'category': 'Title', 'element_id': '56d531394823d81787d77a04462ed096'}, page_content='Lorem ipsum dolor sit amet.')
```


您还可以通过 Unstructured API 在单个 API 中批量处理多个文件，使用 `UnstructuredLoader`。


```python
loader = UnstructuredLoader(
    file_path=["example_data/fake.docx", "example_data/fake-email.eml"],
    api_key=os.getenv("UNSTRUCTURED_API_KEY"),
    partition_via_api=True,
)

docs = loader.load()

print(docs[0].metadata["filename"], ": ", docs[0].page_content[:100])
print(docs[-1].metadata["filename"], ": ", docs[-1].page_content[:100])
```
```output
INFO: Preparing to split document for partition.
INFO: Given file doesn't have '.pdf' extension, so splitting is not enabled.
INFO: Partitioning without split.
INFO: Successfully partitioned the document.
INFO: Preparing to split document for partition.
INFO: Given file doesn't have '.pdf' extension, so splitting is not enabled.
INFO: Partitioning without split.
INFO: Successfully partitioned the document.
``````output
fake.docx :  Lorem ipsum dolor sit amet.
fake-email.eml :  Violets are blue
```

### 非结构化 SDK 客户端

使用非结构化 API 进行分区依赖于 [非结构化 SDK 客户端](https://docs.unstructured.io/api-reference/api-services/sdk)。

以下示例展示了如何自定义客户端的一些功能，使用您自己的 `requests.Session()`，传入替代的 `server_url`，或自定义 `RetryConfig` 对象，以便更好地控制失败请求的处理方式。

请注意，下面的示例可能未使用最新版本的 UnstructuredClient，并且未来版本可能会有破坏性更改。有关最新示例，请参考 [非结构化 Python SDK](https://docs.unstructured.io/api-reference/api-services/sdk-python) 文档。

```python
import requests
from langchain_unstructured import UnstructuredLoader
from unstructured_client import UnstructuredClient
from unstructured_client.utils import BackoffStrategy, RetryConfig

client = UnstructuredClient(
    api_key_auth=os.getenv(
        "UNSTRUCTURED_API_KEY"
    ),  # 注意：客户端 API 参数为 "api_key_auth" 而不是 "api_key"
    client=requests.Session(),
    server_url="https://api.unstructuredapp.io/general/v0/general",
    retry_config=RetryConfig(
        strategy="backoff",
        retry_connection_errors=True,
        backoff=BackoffStrategy(
            initial_interval=500,
            max_interval=60000,
            exponent=1.5,
            max_elapsed_time=900000,
        ),
    ),
)

loader = UnstructuredLoader(
    "./example_data/layout-parser-paper.pdf",
    partition_via_api=True,
    client=client,
)

docs = loader.load()

print(docs[0].metadata["filename"], ": ", docs[0].page_content[:100])
```
```output
INFO: Preparing to split document for partition.
INFO: Concurrency level set to 5
INFO: Splitting pages 1 to 16 (16 total)
INFO: Determined optimal split size of 4 pages.
INFO: Partitioning 4 files with 4 page(s) each.
INFO: Partitioning set #1 (pages 1-4).
INFO: Partitioning set #2 (pages 5-8).
INFO: Partitioning set #3 (pages 9-12).
INFO: Partitioning set #4 (pages 13-16).
INFO: HTTP Request: POST https://api.unstructuredapp.io/general/v0/general "HTTP/1.1 200 OK"
INFO: HTTP Request: POST https://api.unstructuredapp.io/general/v0/general "HTTP/1.1 200 OK"
INFO: HTTP Request: POST https://api.unstructuredapp.io/general/v0/general "HTTP/1.1 200 OK"
INFO: Successfully partitioned set #1, elements added to the final result.
INFO: Successfully partitioned set #2, elements added to the final result.
INFO: Successfully partitioned set #3, elements added to the final result.
INFO: Successfully partitioned set #4, elements added to the final result.
INFO: Successfully partitioned the document.
``````output
layout-parser-paper.pdf :  LayoutParser: A Uniﬁed Toolkit for Deep Learning Based Document Image Analysis
```

## Chunking

`UnstructuredLoader` 不支持 `mode` 作为参数来对文本进行分组，像旧版的加载器 `UnstructuredFileLoader` 和其他加载器那样。它支持“分块”。在非结构化数据中，分块与您可能熟悉的基于纯文本特征形成块的其他分块机制有所不同——例如，像 "\n\n" 或 "\n" 这样的字符序列可能表示段落边界或列表项边界。相反，所有文档都是使用关于每种文档格式的特定知识进行拆分，以将文档划分为语义单元（文档元素），只有在单个元素超过所需的最大块大小时，我们才需要 resort to text-splitting。一般来说，分块会将连续的元素组合在一起，以尽可能大而不超过最大块大小的方式形成块。分块生成一系列 CompositeElement、Table 或 TableChunk 元素。每个“块”都是这三种类型之一的实例。

有关分块选项的更多详细信息，请参见此 [页面](https://docs.unstructured.io/open-source/core-functionality/chunking)，但要重现与 `mode="single"` 相同的行为，您可以设置 `chunking_strategy="basic"`、`max_characters=<some-really-big-number>` 和 `include_orig_elements=False`。


```python
from langchain_unstructured import UnstructuredLoader

loader = UnstructuredLoader(
    "./example_data/layout-parser-paper.pdf",
    chunking_strategy="basic",
    max_characters=1000000,
    include_orig_elements=False,
)

docs = loader.load()

print("Number of LangChain documents:", len(docs))
print("Length of text in the document:", len(docs[0].page_content))
```
```output
WARNING: Partitioning locally even though api_key is defined since partition_via_api=False.
``````output
Number of LangChain documents: 1
Length of text in the document: 42772
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)