---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/elasticsearch_retriever.ipynb
sidebar_label: Elasticsearch
---

# ElasticsearchRetriever

## Overview
>[Elasticsearch](https://www.elastic.co/elasticsearch/) is a distributed, RESTful search and analytics engine. It provides a distributed, multitenant-capable full-text search engine with an HTTP web interface and schema-free JSON documents. It supports keyword search, vector search, hybrid search and complex filtering.

The `ElasticsearchRetriever` is a generic wrapper to enable flexible access to all `Elasticsearch` features through the [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html).  For most use cases the other classes (`ElasticsearchStore`, `ElasticsearchEmbeddings`, etc.) should suffice, but if they don't you can use `ElasticsearchRetriever`.

This guide will help you getting started with the Elasticsearch [retriever](/docs/concepts/#retrievers). For detailed documentation of all `ElasticsearchRetriever` features and configurations head to the [API reference](https://api.python.langchain.com/en/latest/retrievers/langchain_elasticsearch.retrievers.ElasticsearchRetriever.html).

### Integration details

| Retriever | Self-host | Cloud offering | Package |
| :--- | :--- | :---: | :---: |
[ElasticsearchRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain_elasticsearch.retrievers.ElasticsearchRetriever.html) | ✅ | ✅ | langchain_elasticsearch |


## Setup

There are two main ways to set up an Elasticsearch instance:

- Elastic Cloud: [Elastic Cloud](https://cloud.elastic.co/) is a managed Elasticsearch service. Sign up for a [free trial](https://www.elastic.co/cloud/cloud-trial-overview).
To connect to an Elasticsearch instance that does not require login credentials (starting the docker instance with security enabled), pass the Elasticsearch URL and index name along with the embedding object to the constructor.

- Local Install Elasticsearch: Get started with Elasticsearch by running it locally. The easiest way is to use the official Elasticsearch Docker image. See the [Elasticsearch Docker documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html) for more information.

If you want to get automated tracing from individual queries, you can also set your [LangSmith](https://docs.smith.langchain.com/) API key by uncommenting below:


```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### Installation

This retriever lives in the `langchain-elasticsearch` package. For demonstration purposes, we will also install `langchain-community` to generate text [embeddings](/docs/concepts/#embedding-models).


```python
%pip install -qU langchain-community langchain-elasticsearch
```


```python
from typing import Any, Dict, Iterable

from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk
from langchain_community.embeddings import DeterministicFakeEmbedding
from langchain_core.documents import Document
from langchain_core.embeddings import Embeddings
from langchain_elasticsearch import ElasticsearchRetriever
```

### Configure

Here we define the conncection to Elasticsearch. In this example we use a locally running instance. Alternatively, you can make an account in [Elastic Cloud](https://cloud.elastic.co/) and start a [free trial](https://www.elastic.co/cloud/cloud-trial-overview).


```python
es_url = "http://localhost:9200"
es_client = Elasticsearch(hosts=[es_url])
es_client.info()
```

For vector search, we are going to use random embeddings just for illustration. For real use cases, pick one of the available LangChain [Embeddings](/docs/integrations/text_embedding) classes.


```python
embeddings = DeterministicFakeEmbedding(size=3)
```

#### Define example data


```python
index_name = "test-langchain-retriever"
text_field = "text"
dense_vector_field = "fake_embedding"
num_characters_field = "num_characters"
texts = [
    "foo",
    "bar",
    "world",
    "hello world",
    "hello",
    "foo bar",
    "bla bla foo",
]
```

#### Index data

Typically, users make use of `ElasticsearchRetriever` when they already have data in an Elasticsearch index. Here we index some example text documents. If you created an index for example using `ElasticsearchStore.from_documents` that's also fine.


```python
def create_index(
    es_client: Elasticsearch,
    index_name: str,
    text_field: str,
    dense_vector_field: str,
    num_characters_field: str,
):
    es_client.indices.create(
        index=index_name,
        mappings={
            "properties": {
                text_field: {"type": "text"},
                dense_vector_field: {"type": "dense_vector"},
                num_characters_field: {"type": "integer"},
            }
        },
    )


def index_data(
    es_client: Elasticsearch,
    index_name: str,
    text_field: str,
    dense_vector_field: str,
    embeddings: Embeddings,
    texts: Iterable[str],
    refresh: bool = True,
) -> None:
    create_index(
        es_client, index_name, text_field, dense_vector_field, num_characters_field
    )

    vectors = embeddings.embed_documents(list(texts))
    requests = [
        {
            "_op_type": "index",
            "_index": index_name,
            "_id": i,
            text_field: text,
            dense_vector_field: vector,
            num_characters_field: len(text),
        }
        for i, (text, vector) in enumerate(zip(texts, vectors))
    ]

    bulk(es_client, requests)

    if refresh:
        es_client.indices.refresh(index=index_name)

    return len(requests)
```


```python
index_data(es_client, index_name, text_field, dense_vector_field, embeddings, texts)
```



```output
7
```


## Instantiation

### Vector search

Dense vector retrival using fake embeddings in this example.


```python
def vector_query(search_query: str) -> Dict:
    vector = embeddings.embed_query(search_query)  # same embeddings as for indexing
    return {
        "knn": {
            "field": dense_vector_field,
            "query_vector": vector,
            "k": 5,
            "num_candidates": 10,
        }
    }


vector_retriever = ElasticsearchRetriever.from_es_params(
    index_name=index_name,
    body_func=vector_query,
    content_field=text_field,
    url=es_url,
)

vector_retriever.invoke("foo")
```



```output
[Document(page_content='foo', metadata={'_index': 'test-langchain-index', '_id': '0', '_score': 1.0, '_source': {'fake_embedding': [-2.336764233933763, 0.27510289545940503, -0.7957597268194339], 'num_characters': 3}}),
 Document(page_content='world', metadata={'_index': 'test-langchain-index', '_id': '2', '_score': 0.6770179, '_source': {'fake_embedding': [-0.7041151202179595, -1.4652961969276497, -0.25786766898672847], 'num_characters': 5}}),
 Document(page_content='hello world', metadata={'_index': 'test-langchain-index', '_id': '3', '_score': 0.4816144, '_source': {'fake_embedding': [0.42728413221815387, -1.1889908285425348, -1.445433230084671], 'num_characters': 11}}),
 Document(page_content='hello', metadata={'_index': 'test-langchain-index', '_id': '4', '_score': 0.46853775, '_source': {'fake_embedding': [-0.28560441330564046, 0.9958894823084921, 1.5489829880195058], 'num_characters': 5}}),
 Document(page_content='foo bar', metadata={'_index': 'test-langchain-index', '_id': '5', '_score': 0.2086992, '_source': {'fake_embedding': [0.2533670476638539, 0.08100381646160418, 0.7763644080870179], 'num_characters': 7}})]
```


### BM25

Traditional keyword matching.


```python
def bm25_query(search_query: str) -> Dict:
    return {
        "query": {
            "match": {
                text_field: search_query,
            },
        },
    }


bm25_retriever = ElasticsearchRetriever.from_es_params(
    index_name=index_name,
    body_func=bm25_query,
    content_field=text_field,
    url=es_url,
)

bm25_retriever.invoke("foo")
```



```output
[Document(page_content='foo', metadata={'_index': 'test-langchain-index', '_id': '0', '_score': 0.9711467, '_source': {'fake_embedding': [-2.336764233933763, 0.27510289545940503, -0.7957597268194339], 'num_characters': 3}}),
 Document(page_content='foo bar', metadata={'_index': 'test-langchain-index', '_id': '5', '_score': 0.7437035, '_source': {'fake_embedding': [0.2533670476638539, 0.08100381646160418, 0.7763644080870179], 'num_characters': 7}}),
 Document(page_content='bla bla foo', metadata={'_index': 'test-langchain-index', '_id': '6', '_score': 0.6025789, '_source': {'fake_embedding': [1.7365927060137358, -0.5230400847844948, 0.7978339724186192], 'num_characters': 11}})]
```


### Hybrid search

The combination of vector search and BM25 search using [Reciprocal Rank Fusion](https://www.elastic.co/guide/en/elasticsearch/reference/current/rrf.html) (RRF) to combine the result sets.


```python
def hybrid_query(search_query: str) -> Dict:
    vector = embeddings.embed_query(search_query)  # same embeddings as for indexing
    return {
        "query": {
            "match": {
                text_field: search_query,
            },
        },
        "knn": {
            "field": dense_vector_field,
            "query_vector": vector,
            "k": 5,
            "num_candidates": 10,
        },
        "rank": {"rrf": {}},
    }


hybrid_retriever = ElasticsearchRetriever.from_es_params(
    index_name=index_name,
    body_func=hybrid_query,
    content_field=text_field,
    url=es_url,
)

hybrid_retriever.invoke("foo")
```



```output
[Document(page_content='foo', metadata={'_index': 'test-langchain-index', '_id': '0', '_score': 0.9711467, '_source': {'fake_embedding': [-2.336764233933763, 0.27510289545940503, -0.7957597268194339], 'num_characters': 3}}),
 Document(page_content='foo bar', metadata={'_index': 'test-langchain-index', '_id': '5', '_score': 0.7437035, '_source': {'fake_embedding': [0.2533670476638539, 0.08100381646160418, 0.7763644080870179], 'num_characters': 7}}),
 Document(page_content='bla bla foo', metadata={'_index': 'test-langchain-index', '_id': '6', '_score': 0.6025789, '_source': {'fake_embedding': [1.7365927060137358, -0.5230400847844948, 0.7978339724186192], 'num_characters': 11}})]
```


### Fuzzy matching

Keyword matching with typo tolerance.


```python
def fuzzy_query(search_query: str) -> Dict:
    return {
        "query": {
            "match": {
                text_field: {
                    "query": search_query,
                    "fuzziness": "AUTO",
                }
            },
        },
    }


fuzzy_retriever = ElasticsearchRetriever.from_es_params(
    index_name=index_name,
    body_func=fuzzy_query,
    content_field=text_field,
    url=es_url,
)

fuzzy_retriever.invoke("fox")  # note the character tolernace
```



```output
[Document(page_content='foo', metadata={'_index': 'test-langchain-index', '_id': '0', '_score': 0.6474311, '_source': {'fake_embedding': [-2.336764233933763, 0.27510289545940503, -0.7957597268194339], 'num_characters': 3}}),
 Document(page_content='foo bar', metadata={'_index': 'test-langchain-index', '_id': '5', '_score': 0.49580228, '_source': {'fake_embedding': [0.2533670476638539, 0.08100381646160418, 0.7763644080870179], 'num_characters': 7}}),
 Document(page_content='bla bla foo', metadata={'_index': 'test-langchain-index', '_id': '6', '_score': 0.40171927, '_source': {'fake_embedding': [1.7365927060137358, -0.5230400847844948, 0.7978339724186192], 'num_characters': 11}})]
```


### Complex filtering

Combination of filters on different fields.


```python
def filter_query_func(search_query: str) -> Dict:
    return {
        "query": {
            "bool": {
                "must": [
                    {"range": {num_characters_field: {"gte": 5}}},
                ],
                "must_not": [
                    {"prefix": {text_field: "bla"}},
                ],
                "should": [
                    {"match": {text_field: search_query}},
                ],
            }
        }
    }


filtering_retriever = ElasticsearchRetriever.from_es_params(
    index_name=index_name,
    body_func=filter_query_func,
    content_field=text_field,
    url=es_url,
)

filtering_retriever.invoke("foo")
```



```output
[Document(page_content='foo bar', metadata={'_index': 'test-langchain-index', '_id': '5', '_score': 1.7437035, '_source': {'fake_embedding': [0.2533670476638539, 0.08100381646160418, 0.7763644080870179], 'num_characters': 7}}),
 Document(page_content='world', metadata={'_index': 'test-langchain-index', '_id': '2', '_score': 1.0, '_source': {'fake_embedding': [-0.7041151202179595, -1.4652961969276497, -0.25786766898672847], 'num_characters': 5}}),
 Document(page_content='hello world', metadata={'_index': 'test-langchain-index', '_id': '3', '_score': 1.0, '_source': {'fake_embedding': [0.42728413221815387, -1.1889908285425348, -1.445433230084671], 'num_characters': 11}}),
 Document(page_content='hello', metadata={'_index': 'test-langchain-index', '_id': '4', '_score': 1.0, '_source': {'fake_embedding': [-0.28560441330564046, 0.9958894823084921, 1.5489829880195058], 'num_characters': 5}})]
```


Note that the query match is on top. The other documents that got passed the filter are also in the result set, but they all have the same score.

### Custom document mapper

It is possible to cusomize the function tha maps an Elasticsearch result (hit) to a LangChain document.


```python
def num_characters_mapper(hit: Dict[str, Any]) -> Document:
    num_chars = hit["_source"][num_characters_field]
    content = hit["_source"][text_field]
    return Document(
        page_content=f"This document has {num_chars} characters",
        metadata={"text_content": content},
    )


custom_mapped_retriever = ElasticsearchRetriever.from_es_params(
    index_name=index_name,
    body_func=filter_query_func,
    document_mapper=num_characters_mapper,
    url=es_url,
)

custom_mapped_retriever.invoke("foo")
```



```output
[Document(page_content='This document has 7 characters', metadata={'text_content': 'foo bar'}),
 Document(page_content='This document has 5 characters', metadata={'text_content': 'world'}),
 Document(page_content='This document has 11 characters', metadata={'text_content': 'hello world'}),
 Document(page_content='This document has 5 characters', metadata={'text_content': 'hello'})]
```


## Usage

Following the above examples, we use `.invoke` to issue a single query. Because retrievers are Runnables, we can use any method in the [Runnable interface](/docs/concepts/#runnable-interface), such as `.batch`, as well.

## Use within a chain

We can also incorporate retrievers into [chains](/docs/how_to/sequence/) to build larger applications, such as a simple [RAG](/docs/tutorials/rag/) application. For demonstration purposes, we instantiate an OpenAI chat model as well.


```python
%pip install -qU langchain-openai
```


```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the context provided.

Context: {context}

Question: {question}"""
)

llm = ChatOpenAI(model="gpt-3.5-turbo-0125")


def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)


chain = (
    {"context": vector_retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
```


```python
chain.invoke("what is foo?")
```

## API reference

For detailed documentation of all `ElasticsearchRetriever` features and configurations head to the [API reference](https://api.python.langchain.com/en/latest/retrievers/langchain_elasticsearch.retrievers.ElasticsearchRetriever.html).


## Related

- Retriever [conceptual guide](/docs/concepts/#retrievers)
- Retriever [how-to guides](/docs/how_to/#retrievers)