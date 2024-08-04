---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/unstructured_file.ipynb
---
# Unstructured

This notebook covers how to use `Unstructured` package to load files of many types. `Unstructured` currently supports loading of text files, powerpoints, html, pdfs, images, and more.

Please see [this guide](/docs/integrations/providers/unstructured/) for more instructions on setting up Unstructured locally, including setting up required system dependencies.


```python
# Install package, compatible with API partitioning
%pip install --upgrade --quiet "langchain-unstructured"
```

### Local Partitioning (Optional)

By default, `langchain-unstructured` installs a smaller footprint that requires
offloading of the partitioning logic to the Unstructured API, which requires an `api_key`. For
partitioning using the API, refer to the Unstructured API section below.

If you would like to run the partitioning logic locally, you will need to install
a combination of system dependencies, as outlined in the 
[Unstructured documentation here](https://docs.unstructured.io/open-source/installation/full-installation).

For example, on Macs you can install the required dependencies with:

```bash
# base dependencies
brew install libmagic poppler tesseract

# If parsing xml / html documents:
brew install libxml2 libxslt
```

You can install the required `pip` dependencies with:

```bash
pip install "langchain-unstructured[local]"
```

### Quickstart

To simply load a file as a document, you can use the LangChain `DocumentLoader.load` 
interface:


```python
from langchain_unstructured import UnstructuredLoader

loader = UnstructuredLoader("./example_data/state_of_the_union.txt")

docs = loader.load()
```

### Load list of files


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
## PDF Example

Processing PDF documents works exactly the same way. Unstructured detects the file type and extracts the same types of elements.

### Define a Partitioning Strategy

Unstructured document loader allow users to pass in a `strategy` parameter that lets Unstructured
know how to partition pdf and other OCR'd documents. Currently supported strategies are `"auto"`,
`"hi_res"`, `"ocr_only"`, and `"fast"`. Learn more about the different strategies
[here](https://docs.unstructured.io/open-source/core-functionality/partitioning#partition-pdf). 

Not all document types have separate hi res and fast partitioning strategies. For those document types, the `strategy` kwarg is
ignored. In some cases, the high res strategy will fallback to fast if there is a dependency missing
(i.e. a model for document partitioning). You can see how to apply a strategy to an
`UnstructuredLoader` below.


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


## Post Processing

If you need to post process the `unstructured` elements after extraction, you can pass in a list of
`str` -> `str` functions to the `post_processors` kwarg when you instantiate the `UnstructuredLoader`. This applies to other Unstructured loaders as well. Below is an example.


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

If you want to get up and running with smaller packages and get the most up-to-date partitioning you can `pip install
unstructured-client` and `pip install langchain-unstructured`. For
more information about the `UnstructuredLoader`, refer to the
[Unstructured provider page](https://python.langchain.com/v0.1/docs/integrations/document_loaders/unstructured_file/).

The loader will process your document using the hosted Unstructured serverless API when you pass in
your `api_key` and set `partition_via_api=True`. You can generate a free
Unstructured API key [here](https://unstructured.io/api-key/).

Check out the instructions [here](https://github.com/Unstructured-IO/unstructured-api#dizzy-instructions-for-using-the-docker-image)
if you’d like to self-host the Unstructured API or run it locally.


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


You can also batch multiple files through the Unstructured API in a single API using `UnstructuredLoader`.


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
### Unstructured SDK Client

Partitioning with the Unstructured API relies on the [Unstructured SDK
Client](https://docs.unstructured.io/api-reference/api-services/sdk).

Below is an example showing how you can customize some features of the client and use your own `requests.Session()`, pass in an alternative `server_url`, or customize the `RetryConfig` object for more control over how failed requests are handled.

Note that the example below may not use the latest version of the UnstructuredClient and there could be breaking changes in future releases. For the latest examples, refer to the [Unstructured Python SDK](https://docs.unstructured.io/api-reference/api-services/sdk-python) docs.


```python
import requests
from langchain_unstructured import UnstructuredLoader
from unstructured_client import UnstructuredClient
from unstructured_client.utils import BackoffStrategy, RetryConfig

client = UnstructuredClient(
    api_key_auth=os.getenv(
        "UNSTRUCTURED_API_KEY"
    ),  # Note: the client API param is "api_key_auth" instead of "api_key"
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

The `UnstructuredLoader` does not support `mode` as parameter for grouping text like the older
loader `UnstructuredFileLoader` and others did. It instead supports "chunking". Chunking in
unstructured differs from other chunking mechanisms you may be familiar with that form chunks based
on plain-text features--character sequences like "\n\n" or "\n" that might indicate a paragraph
boundary or list-item boundary. Instead, all documents are split using specific knowledge about each
document format to partition the document into semantic units (document elements) and we only need to
resort to text-splitting when a single element exceeds the desired maximum chunk size. In general,
chunking combines consecutive elements to form chunks as large as possible without exceeding the
maximum chunk size. Chunking produces a sequence of CompositeElement, Table, or TableChunk elements.
Each “chunk” is an instance of one of these three types.

See this [page](https://docs.unstructured.io/open-source/core-functionality/chunking) for more
details about chunking options, but to reproduce the same behavior as `mode="single"`, you can set
`chunking_strategy="basic"`, `max_characters=<some-really-big-number>`, and `include_orig_elements=False`.


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

## Related

- Document loader [conceptual guide](/docs/concepts/#document-loaders)
- Document loader [how-to guides](/docs/how_to/#document-loaders)
