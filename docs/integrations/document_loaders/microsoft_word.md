---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/microsoft_word.ipynb
---
# Microsoft Word

>[Microsoft Word](https://www.microsoft.com/en-us/microsoft-365/word) is a word processor developed by Microsoft.

This covers how to load `Word` documents into a document format that we can use downstream.

## Using Docx2txt

Load .docx using `Docx2txt` into a document.


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


## Using Unstructured

Please see [this guide](/docs/integrations/providers/unstructured/) for more instructions on setting up Unstructured locally, including setting up required system dependencies.


```python
from langchain_community.document_loaders import UnstructuredWordDocumentLoader

loader = UnstructuredWordDocumentLoader("example_data/fake.docx")

data = loader.load()

data
```



```output
[Document(page_content='Lorem ipsum dolor sit amet.', metadata={'source': 'example_data/fake.docx'})]
```


### Retain Elements

Under the hood, Unstructured creates different "elements" for different chunks of text. By default we combine those together, but you can easily keep that separation by specifying `mode="elements"`.


```python
loader = UnstructuredWordDocumentLoader("./example_data/fake.docx", mode="elements")

data = loader.load()

data[0]
```



```output
Document(page_content='Lorem ipsum dolor sit amet.', metadata={'source': './example_data/fake.docx', 'category_depth': 0, 'file_directory': './example_data', 'filename': 'fake.docx', 'last_modified': '2023-12-19T13:42:18', 'languages': ['por', 'cat'], 'filetype': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document', 'category': 'Title'})
```


## Using Azure AI Document Intelligence

>[Azure AI Document Intelligence](https://aka.ms/doc-intelligence) (formerly known as `Azure Form Recognizer`) is machine-learning 
>based service that extracts texts (including handwriting), tables, document structures (e.g., titles, section headings, etc.) and key-value-pairs from
>digital or scanned PDFs, images, Office and HTML files.
>
>Document Intelligence supports `PDF`, `JPEG/JPG`, `PNG`, `BMP`, `TIFF`, `HEIF`, `DOCX`, `XLSX`, `PPTX` and `HTML`.

This current implementation of a loader using `Document Intelligence` can incorporate content page-wise and turn it into LangChain documents. The default output format is markdown, which can be easily chained with `MarkdownHeaderTextSplitter` for semantic document chunking. You can also use `mode="single"` or `mode="page"` to return pure texts in a single page or document split by page.


## Prerequisite

An Azure AI Document Intelligence resource in one of the 3 preview regions: **East US**, **West US2**, **West Europe** - follow [this document](https://learn.microsoft.com/azure/ai-services/document-intelligence/create-document-intelligence-resource?view=doc-intel-4.0.0) to create one if you don't have. You will be passing `<endpoint>` and `<key>` as parameters to the loader.

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


## Related

- Document loader [conceptual guide](/docs/concepts/#document-loaders)
- Document loader [how-to guides](/docs/how_to/#document-loaders)