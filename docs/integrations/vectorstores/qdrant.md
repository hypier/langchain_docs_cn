---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/qdrant.ipynb
---
# Qdrant

>[Qdrant](https://qdrant.tech/documentation/) (read: quadrant ) is a vector similarity search engine. It provides a production-ready service with a convenient API to store, search, and manage vectors with additional payload and extended filtering support. It makes it useful for all sorts of neural network or semantic-based matching, faceted search, and other applications.

This documentation demonstrates how to use Qdrant with Langchain for dense/sparse and hybrid retrieval.

> This page documents the `QdrantVectorStore` class that supports multiple retrieval modes via Qdrant's new [Query API](https://qdrant.tech/blog/qdrant-1.10.x/). It requires you to run Qdrant v1.10.0 or above.

There are various modes of how to run `Qdrant`, and depending on the chosen one, there will be some subtle differences. The options include:
- Local mode, no server required
- Docker deployments
- Qdrant Cloud

See the [installation instructions](https://qdrant.tech/documentation/install/).


```python
%pip install langchain-qdrant langchain-openai langchain
```

We will use `OpenAIEmbeddings` for demonstration.


```python
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_qdrant import QdrantVectorStore
from langchain_text_splitters import CharacterTextSplitter
```


```python
loader = TextLoader("some-file.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```

## Connecting to Qdrant from LangChain

### Local mode

Python client allows you to run the same code in local mode without running the Qdrant server. That's great for testing things out and debugging or storing just a small amount of vectors. The embeddings might be fully kept in memory or persisted on disk.

#### In-memory

For some testing scenarios and quick experiments, you may prefer to keep all the data in memory only, so it gets lost when the client is destroyed - usually at the end of your script/notebook.


```python
qdrant = QdrantVectorStore.from_documents(
    docs,
    embeddings,
    location=":memory:",  # Local mode with in-memory storage only
    collection_name="my_documents",
)
```

#### On-disk storage

Local mode, without using the Qdrant server, may also store your vectors on disk so they persist between runs.


```python
qdrant = QdrantVectorStore.from_documents(
    docs,
    embeddings,
    path="/tmp/local_qdrant",
    collection_name="my_documents",
)
```

### On-premise server deployment

No matter if you choose to launch Qdrant locally with [a Docker container](https://qdrant.tech/documentation/install/), or select a Kubernetes deployment with [the official Helm chart](https://github.com/qdrant/qdrant-helm), the way you're going to connect to such an instance will be identical. You'll need to provide a URL pointing to the service.


```python
url = "<---qdrant url here --->"
qdrant = QdrantVectorStore.from_documents(
    docs,
    embeddings,
    url=url,
    prefer_grpc=True,
    collection_name="my_documents",
)
```

### Qdrant Cloud

If you prefer not to keep yourself busy with managing the infrastructure, you can choose to set up a fully-managed Qdrant cluster on [Qdrant Cloud](https://cloud.qdrant.io/). There is a free forever 1GB cluster included for trying out. The main difference with using a managed version of Qdrant is that you'll need to provide an API key to secure your deployment from being accessed publicly. The value can also be set in a `QDRANT_API_KEY` environment variable.


```python
url = "<---qdrant cloud cluster url here --->"
api_key = "<---api key here--->"
qdrant = QdrantVectorStore.from_documents(
    docs,
    embeddings,
    url=url,
    prefer_grpc=True,
    api_key=api_key,
    collection_name="my_documents",
)
```

## Using an existing collection

To get an instance of `langchain_qdrant.Qdrant` without loading any new documents or texts, you can use the `Qdrant.from_existing_collection()` method.


```python
qdrant = QdrantVectorStore.from_existing_collection(
    embeddings=embeddings,
    collection_name="my_documents",
    url="http://localhost:6333",
)
```

## Recreating the collection

The collection is reused if it already exists. Setting `force_recreate` to `True` allows to remove the old collection and start from scratch.


```python
url = "<---qdrant url here --->"
qdrant = QdrantVectorStore.from_documents(
    docs,
    embeddings,
    url=url,
    prefer_grpc=True,
    collection_name="my_documents",
    force_recreate=True,
)
```

## Similarity search

The simplest scenario for using Qdrant vector store is to perform a similarity search. Under the hood, our query will be encoded into vector embeddings and used to find similar documents in Qdrant collection.

`QdrantVectorStore` supports 3 modes for similarity searches. They can be configured using the `retrieval_mode` parameter when setting up the class.

- Dense Vector Search(Default)
- Sparse Vector Search
- Hybrid Search

### Dense Vector Search

To search with only dense vectors,

- The `retrieval_mode` parameter should be set to `RetrievalMode.DENSE`(default).
- A [dense embeddings](https://python.langchain.com/v0.2/docs/integrations/text_embedding/) value should be provided to the `embedding` parameter.


```python
from langchain_qdrant import RetrievalMode

qdrant = QdrantVectorStore.from_documents(
    docs,
    embedding=embeddings,
    location=":memory:",
    collection_name="my_documents",
    retrieval_mode=RetrievalMode.DENSE,
)

query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.similarity_search(query)
```

### Sparse Vector Search

To search with only sparse vectors,

- The `retrieval_mode` parameter should be set to `RetrievalMode.SPARSE`.
- An implementation of the [`SparseEmbeddings`](https://github.com/langchain-ai/langchain/blob/master/libs/partners/qdrant/langchain_qdrant/sparse_embeddings.py) interface using any sparse embeddings provider has to be provided as value to the `sparse_embedding` parameter.

The `langchain-qdrant` package provides a [FastEmbed](https://github.com/qdrant/fastembed) based implementation out of the box.

To use it, install the FastEmbed package.


```python
%pip install fastembed
```


```python
from langchain_qdrant import FastEmbedSparse, RetrievalMode

sparse_embeddings = FastEmbedSparse(model_name="Qdrant/BM25")

qdrant = QdrantVectorStore.from_documents(
    docs,
    sparse_embedding=sparse_embeddings,
    location=":memory:",
    collection_name="my_documents",
    retrieval_mode=RetrievalMode.SPARSE,
)

query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.similarity_search(query)
```

### Hybrid Vector Search

To perform a hybrid search using dense and sparse vectors with score fusion,

- The `retrieval_mode` parameter should be set to `RetrievalMode.HYBRID`.
- A [dense embeddings](https://python.langchain.com/v0.2/docs/integrations/text_embedding/) value should be provided to the `embedding` parameter.
- An implementation of the [`SparseEmbeddings`](https://github.com/langchain-ai/langchain/blob/master/libs/partners/qdrant/langchain_qdrant/sparse_embeddings.py) interface using any sparse embeddings provider has to be provided as value to the `sparse_embedding` parameter.

Note that if you've added documents with the `HYBRID` mode, you can switch to any retrieval mode when searching. Since both the dense and sparse vectors are available in the collection.


```python
from langchain_qdrant import FastEmbedSparse, RetrievalMode

sparse_embeddings = FastEmbedSparse(model_name="Qdrant/BM25")

qdrant = QdrantVectorStore.from_documents(
    docs,
    embedding=embeddings,
    sparse_embedding=sparse_embeddings,
    location=":memory:",
    collection_name="my_documents",
    retrieval_mode=RetrievalMode.HYBRID,
)

query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.similarity_search(query)
```

## Similarity search with score

Sometimes we might want to perform the search, but also obtain a relevancy score to know how good is a particular result. 
The returned distance score is cosine distance. Therefore, a lower score is better.


```python
query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.similarity_search_with_score(query)
```


```python
document, score = found_docs[0]
print(document.page_content)
print(f"\nScore: {score}")
```

### Metadata filtering

Qdrant has an [extensive filtering system](https://qdrant.tech/documentation/concepts/filtering/) with rich type support. It is also possible to use the filters in Langchain, by passing an additional param to both the `similarity_search_with_score` and `similarity_search` methods.

```python
from qdrant_client.http import models

query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.similarity_search_with_score(query, filter=models.Filter(...))
```

## Maximum marginal relevance search (MMR)

If you'd like to look up some similar documents, but you'd also like to receive diverse results, MMR is the method you should consider. Maximal marginal relevance optimizes for similarity to query AND diversity among selected documents.

Note that MMR search is only available if you've added documents with `DENSE` or `HYBRID` modes. Since it requires dense vectors.


```python
query = "What did the president say about Ketanji Brown Jackson"
found_docs = qdrant.max_marginal_relevance_search(query, k=2, fetch_k=10)
```


```python
for i, doc in enumerate(found_docs):
    print(f"{i + 1}.", doc.page_content, "\n")
```

## Qdrant as a Retriever

Qdrant, as all the other vector stores, is a LangChain Retriever. 


```python
retriever = qdrant.as_retriever()
```

It might be also specified to use MMR as a search strategy, instead of similarity.


```python
retriever = qdrant.as_retriever(search_type="mmr")
```


```python
query = "What did the president say about Ketanji Brown Jackson"
retriever.invoke(query)[0]
```

## Customizing Qdrant

There are options to use an existing Qdrant collection within your Langchain application. In such cases, you may need to define how to map Qdrant point into the Langchain `Document`.

### Named vectors

Qdrant supports [multiple vectors per point](https://qdrant.tech/documentation/concepts/collections/#collection-with-multiple-vectors) by named vectors. If you work with a collection created externally or want to have the differently named vector used, you can configure it by providing its name.



```python
QdrantVectorStore.from_documents(
    docs,
    embedding=embeddings,
    sparse_embedding=sparse_embeddings,
    location=":memory:",
    collection_name="my_documents_2",
    retrieval_mode=RetrievalMode.HYBRID,
    vector_name="custom_vector",
    sparse_vector_name="custom_sparse_vector",
)
```

### Metadata

Qdrant stores your vector embeddings along with the optional JSON-like payload. Payloads are optional, but since LangChain assumes the embeddings are generated from the documents, we keep the context data, so you can extract the original texts as well.

By default, your document is going to be stored in the following payload structure:

```json
{
    "page_content": "Lorem ipsum dolor sit amet",
    "metadata": {
        "foo": "bar"
    }
}
```

You can, however, decide to use different keys for the page content and metadata. That's useful if you already have a collection that you'd like to reuse.


```python
QdrantVectorStore.from_documents(
    docs,
    embeddings,
    location=":memory:",
    collection_name="my_documents_2",
    content_payload_key="my_page_content_key",
    metadata_payload_key="my_meta",
)
```


## Related

- Vector store [conceptual guide](/docs/concepts/#vector-stores)
- Vector store [how-to guides](/docs/how_to/#vector-stores)
