---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/document_loaders/pebblo.ipynb
---
# Pebblo Safe DocumentLoader

> [Pebblo](https://daxa-ai.github.io/pebblo/) enables developers to safely load data and promote their Gen AI app to deployment without worrying about the organization’s compliance and security requirements. The project identifies semantic topics and entities found in the loaded data and summarizes them on the UI or a PDF report.

Pebblo has two components.

1. Pebblo Safe DocumentLoader for Langchain
1. Pebblo Server

This document describes how to augment your existing Langchain DocumentLoader with Pebblo Safe DocumentLoader to get deep data visibility on the types of Topics and Entities ingested into the Gen-AI Langchain application. For details on `Pebblo Server` see this [pebblo server](https://daxa-ai.github.io/pebblo/daemon) document.

Pebblo Safeloader enables safe data ingestion for Langchain `DocumentLoader`. This is done by wrapping the document loader call with `Pebblo Safe DocumentLoader`.

Note: To configure pebblo server on some url other that pebblo's default (localhost:8000) url, put the correct URL in `PEBBLO_CLASSIFIER_URL` env variable. This is configurable using the `classifier_url` keyword argument as well. Ref: [server-configurations](https://daxa-ai.github.io/pebblo/config)

#### How to Pebblo enable Document Loading?

Assume a Langchain RAG application snippet using `CSVLoader` to read a CSV document for inference.

Here is the snippet of Document loading using `CSVLoader`.


```python
from langchain_community.document_loaders import CSVLoader

loader = CSVLoader("data/corp_sens_data.csv")
documents = loader.load()
print(documents)
```

The Pebblo SafeLoader can be enabled with few lines of code change to the above snippet.


```python
from langchain_community.document_loaders import CSVLoader, PebbloSafeLoader

loader = PebbloSafeLoader(
    CSVLoader("data/corp_sens_data.csv"),
    name="acme-corp-rag-1",  # App name (Mandatory)
    owner="Joe Smith",  # Owner (Optional)
    description="Support productivity RAG application",  # Description (Optional)
)
documents = loader.load()
print(documents)
```

### Send semantic topics and identities to Pebblo cloud server

To send semantic data to pebblo-cloud, pass api-key to PebbloSafeLoader as an argument or alternatively, put the api-key in `PEBBLO_API_KEY` environment variable.


```python
from langchain_community.document_loaders import CSVLoader, PebbloSafeLoader

loader = PebbloSafeLoader(
    CSVLoader("data/corp_sens_data.csv"),
    name="acme-corp-rag-1",  # App name (Mandatory)
    owner="Joe Smith",  # Owner (Optional)
    description="Support productivity RAG application",  # Description (Optional)
    api_key="my-api-key",  # API key (Optional, can be set in the environment variable PEBBLO_API_KEY)
)
documents = loader.load()
print(documents)
```

### Add semantic topics and identities to loaded metadata

To add semantic topics and sematic entities to metadata of loaded documents, set load_semantic to True as an argument or alternatively, define a new environment variable `PEBBLO_LOAD_SEMANTIC`, and setting it to True.


```python
from langchain_community.document_loaders import CSVLoader, PebbloSafeLoader

loader = PebbloSafeLoader(
    CSVLoader("data/corp_sens_data.csv"),
    name="acme-corp-rag-1",  # App name (Mandatory)
    owner="Joe Smith",  # Owner (Optional)
    description="Support productivity RAG application",  # Description (Optional)
    api_key="my-api-key",  # API key (Optional, can be set in the environment variable PEBBLO_API_KEY)
    load_semantic=True,  # Load semantic data (Optional, default is False, can be set in the environment variable PEBBLO_LOAD_SEMANTIC)
)
documents = loader.load()
print(documents[0].metadata)
```


## Related

- Document loader [conceptual guide](/docs/concepts/#document-loaders)
- Document loader [how-to guides](/docs/how_to/#document-loaders)