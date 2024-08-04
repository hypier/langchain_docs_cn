---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/databricks_vector_search.ipynb
---
# Databricks Vector Search

Databricks Vector Search is a serverless similarity search engine that allows you to store a vector representation of your data, including metadata, in a vector database. With Vector Search, you can create auto-updating vector search indexes from Delta tables managed by Unity Catalog and query them with a simple API to return the most similar vectors.

This notebook shows how to use LangChain with Databricks Vector Search.

Install `databricks-vectorsearch` and related Python packages used in this notebook.


```python
%pip install --upgrade --quiet  langchain-core databricks-vectorsearch langchain-openai tiktoken
```

Use `OpenAIEmbeddings` for the embeddings.


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

Split documents and get embeddings.


```python
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
emb_dim = len(embeddings.embed_query("hello"))
```

## Setup Databricks Vector Search client


```python
from databricks.vector_search.client import VectorSearchClient

vsc = VectorSearchClient()
```

## Create a Vector Search Endpoint
This endpoint is used to create and access vector search indexes.


```python
vsc.create_endpoint(name="vector_search_demo_endpoint", endpoint_type="STANDARD")
```

## Create Direct Vector Access Index
Direct Vector Access Index supports direct read and write of embedding vectors and metadata through a REST API or an SDK. For this index, you manage embedding vectors and index updates yourself.


```python
vector_search_endpoint_name = "vector_search_demo_endpoint"
index_name = "ml.llm.demo_index"

index = vsc.create_direct_access_index(
    endpoint_name=vector_search_endpoint_name,
    index_name=index_name,
    primary_key="id",
    embedding_dimension=emb_dim,
    embedding_vector_column="text_vector",
    schema={
        "id": "string",
        "text": "string",
        "text_vector": "array<float>",
        "source": "string",
    },
)

index.describe()
```


```python
from langchain_community.vectorstores import DatabricksVectorSearch

dvs = DatabricksVectorSearch(
    index, text_column="text", embedding=embeddings, columns=["source"]
)
```

## Add docs to the index


```python
dvs.add_documents(docs)
```

## Similarity search
Optional keyword arguments to similarity_search include specifying k number of documents to retrive, 
a filters dictionary for metadata filtering based on [this syntax](https://docs.databricks.com/en/generative-ai/create-query-vector-search.html#use-filters-on-queries),
as well as the [query_type](https://api-docs.databricks.com/python/vector-search/databricks.vector_search.html#databricks.vector_search.index.VectorSearchIndex.similarity_search) which can be ANN or HYBRID 


```python
query = "What did the president say about Ketanji Brown Jackson"
dvs.similarity_search(query)
print(docs[0].page_content)
```

## Work with Delta Sync Index

You can also use `DatabricksVectorSearch` to search in a Delta Sync Index. Delta Sync Index automatically syncs from a Delta table. You don't need to call `add_text`/`add_documents` manually. See [Databricks documentation page](https://docs.databricks.com/en/generative-ai/vector-search.html#delta-sync-index-with-managed-embeddings) for more details.


```python
dvs_delta_sync = DatabricksVectorSearch("catalog_name.schema_name.delta_sync_index")
dvs_delta_sync.similarity_search(query)
```


## Related

- Vector store [conceptual guide](/docs/concepts/#vector-stores)
- Vector store [how-to guides](/docs/how_to/#vector-stores)
