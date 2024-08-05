---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/microsoft_word.ipynb
---

# Microsoft Word

>[Microsoft Word](https://www.microsoft.com/en-us/microsoft-365/word) 是由微软开发的文字处理软件。

这部分内容涵盖了如何将 `Word` 文档加载到我们可以在后续使用的文档格式中。

## 使用 Docx2txt

使用 `Docx2txt` 加载 .docx 文件到文档中。

```python
%pip install --upgrade --quiet  docx2txt
```

```python
from langchain_community.document_loaders import Docx2txtLoader

loader = Docx2txtLoader("./example_data/fake.docx")

data = loader.load()

data
```

```output
[Document(page_content='Lorem ipsum dolor sit amet.', metadata={'source': './example_data/fake.docx'})]
```

## 使用 Unstructured

请参阅 [此指南](/docs/integrations/providers/unstructured/) 以获取有关本地设置 Unstructured 的更多说明，包括设置所需的系统依赖项。

```python
from langchain_community.document_loaders import UnstructuredWordDocumentLoader

loader = UnstructuredWordDocumentLoader("example_data/fake.docx")

data = loader.load()

data
```

```output
[Document(page_content='Lorem ipsum dolor sit amet.', metadata={'source': 'example_data/fake.docx'})]
```

### 保留元素

在后台，Unstructured 为不同的文本块创建不同的“元素”。默认情况下，我们将这些元素组合在一起，但您可以通过指定 `mode="elements"` 来轻松保持这种分离。

```python
loader = UnstructuredWordDocumentLoader("./example_data/fake.docx", mode="elements")

data = loader.load()

data[0]
```

```output
Document(page_content='Lorem ipsum dolor sit amet.', metadata={'source': './example_data/fake.docx', 'category_depth': 0, 'file_directory': './example_data', 'filename': 'fake.docx', 'last_modified': '2023-12-19T13:42:18', 'languages': ['por', 'cat'], 'filetype': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document', 'category': 'Title'})
```

## 使用 Azure AI 文档智能

>[Azure AI 文档智能](https://aka.ms/doc-intelligence)（以前称为 `Azure Form Recognizer`）是一种基于机器学习的服务，可以从数字或扫描的 PDF、图像、Office 和 HTML 文件中提取文本（包括手写文本）、表格、文档结构（例如标题、章节标题等）和键值对。

>文档智能支持 `PDF`、`JPEG/JPG`、`PNG`、`BMP`、`TIFF`、`HEIF`、`DOCX`、`XLSX`、`PPTX` 和 `HTML`。

当前使用 `文档智能` 的加载器实现可以逐页整合内容并将其转换为 LangChain 文档。默认输出格式为 markdown，可以轻松与 `MarkdownHeaderTextSplitter` 链接，用于语义文档分块。您还可以使用 `mode="single"` 或 `mode="page"` 返回单页或按页分割的纯文本。

## 前提条件

在以下 3 个预览区域之一创建 Azure AI Document Intelligence 资源：**东部美国**、**西部美国 2**、**西欧** - 如果您还没有，请按照 [此文档](https://learn.microsoft.com/azure/ai-services/document-intelligence/create-document-intelligence-resource?view=doc-intel-4.0.0) 创建一个。您将把 `<endpoint>` 和 `<key>` 作为参数传递给加载器。

%pip install --upgrade --quiet  langchain langchain-community azure-ai-documentintelligence


```python
from langchain_community.document_loaders import AzureAIDocumentIntelligenceLoader

file_path = "<filepath>"
endpoint = "<endpoint>"
key = "<key>"
loader = AzureAIDocumentIntelligenceLoader(
    api_endpoint=endpoint, api_key=key, file_path=file_path, api_model="prebuilt-layout"
)

documents = loader.load()
```

## 相关

- 文档加载器 [概念指南](/docs/concepts/#document-loaders)
- 文档加载器 [操作指南](/docs/how_to/#document-loaders)