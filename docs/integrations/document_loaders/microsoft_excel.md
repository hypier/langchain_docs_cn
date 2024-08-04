---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/microsoft_excel.ipynb
---
# Microsoft Excel

The `UnstructuredExcelLoader` is used to load `Microsoft Excel` files. The loader works with both `.xlsx` and `.xls` files. The page content will be the raw text of the Excel file. If you use the loader in `"elements"` mode, an HTML representation of the Excel file will be available in the document metadata under the `text_as_html` key.

Please see [this guide](/docs/integrations/providers/unstructured/) for more instructions on setting up Unstructured locally, including setting up required system dependencies.


```python
%pip install --upgrade --quiet langchain-community unstructured openpyxl
```


```python
from langchain_community.document_loaders import UnstructuredExcelLoader

loader = UnstructuredExcelLoader("./example_data/stanley-cups.xlsx", mode="elements")
docs = loader.load()

print(len(docs))

docs
```
```output
4
```


```output
[Document(page_content='Stanley Cups', metadata={'source': './example_data/stanley-cups.xlsx', 'file_directory': './example_data', 'filename': 'stanley-cups.xlsx', 'last_modified': '2023-12-19T13:42:18', 'page_name': 'Stanley Cups', 'page_number': 1, 'languages': ['eng'], 'filetype': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet', 'category': 'Title'}),
 Document(page_content='\n\n\nTeam\nLocation\nStanley Cups\n\n\nBlues\nSTL\n1\n\n\nFlyers\nPHI\n2\n\n\nMaple Leafs\nTOR\n13\n\n\n', metadata={'source': './example_data/stanley-cups.xlsx', 'file_directory': './example_data', 'filename': 'stanley-cups.xlsx', 'last_modified': '2023-12-19T13:42:18', 'page_name': 'Stanley Cups', 'page_number': 1, 'text_as_html': '<table border="1" class="dataframe">\n  <tbody>\n    <tr>\n      <td>Team</td>\n      <td>Location</td>\n      <td>Stanley Cups</td>\n    </tr>\n    <tr>\n      <td>Blues</td>\n      <td>STL</td>\n      <td>1</td>\n    </tr>\n    <tr>\n      <td>Flyers</td>\n      <td>PHI</td>\n      <td>2</td>\n    </tr>\n    <tr>\n      <td>Maple Leafs</td>\n      <td>TOR</td>\n      <td>13</td>\n    </tr>\n  </tbody>\n</table>', 'languages': ['eng'], 'parent_id': '17e9a90f9616f2abed8cf32b5bd3810d', 'filetype': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet', 'category': 'Table'}),
 Document(page_content='Stanley Cups Since 67', metadata={'source': './example_data/stanley-cups.xlsx', 'file_directory': './example_data', 'filename': 'stanley-cups.xlsx', 'last_modified': '2023-12-19T13:42:18', 'page_name': 'Stanley Cups Since 67', 'page_number': 2, 'languages': ['eng'], 'filetype': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet', 'category': 'Title'}),
 Document(page_content='\n\n\nTeam\nLocation\nStanley Cups\n\n\nBlues\nSTL\n1\n\n\nFlyers\nPHI\n2\n\n\nMaple Leafs\nTOR\n0\n\n\n', metadata={'source': './example_data/stanley-cups.xlsx', 'file_directory': './example_data', 'filename': 'stanley-cups.xlsx', 'last_modified': '2023-12-19T13:42:18', 'page_name': 'Stanley Cups Since 67', 'page_number': 2, 'text_as_html': '<table border="1" class="dataframe">\n  <tbody>\n    <tr>\n      <td>Team</td>\n      <td>Location</td>\n      <td>Stanley Cups</td>\n    </tr>\n    <tr>\n      <td>Blues</td>\n      <td>STL</td>\n      <td>1</td>\n    </tr>\n    <tr>\n      <td>Flyers</td>\n      <td>PHI</td>\n      <td>2</td>\n    </tr>\n    <tr>\n      <td>Maple Leafs</td>\n      <td>TOR</td>\n      <td>0</td>\n    </tr>\n  </tbody>\n</table>', 'languages': ['eng'], 'parent_id': 'ee34bd8c186b57e3530d5443ffa58122', 'filetype': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet', 'category': 'Table'})]
```


## Using Azure AI Document Intelligence

>[Azure AI Document Intelligence](https://aka.ms/doc-intelligence) (formerly known as `Azure Form Recognizer`) is machine-learning 
>based service that extracts texts (including handwriting), tables, document structures (e.g., titles, section headings, etc.) and key-value-pairs from
>digital or scanned PDFs, images, Office and HTML files.
>
>Document Intelligence supports `PDF`, `JPEG/JPG`, `PNG`, `BMP`, `TIFF`, `HEIF`, `DOCX`, `XLSX`, `PPTX` and `HTML`.

This current implementation of a loader using `Document Intelligence` can incorporate content page-wise and turn it into LangChain documents. The default output format is markdown, which can be easily chained with `MarkdownHeaderTextSplitter` for semantic document chunking. You can also use `mode="single"` or `mode="page"` to return pure texts in a single page or document split by page.


### Prerequisite

An Azure AI Document Intelligence resource in one of the 3 preview regions: **East US**, **West US2**, **West Europe** - follow [this document](https://learn.microsoft.com/azure/ai-services/document-intelligence/create-document-intelligence-resource?view=doc-intel-4.0.0) to create one if you don't have. You will be passing `<endpoint>` and `<key>` as parameters to the loader.


```python
%pip install --upgrade --quiet langchain langchain-community azure-ai-documentintelligence
```


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