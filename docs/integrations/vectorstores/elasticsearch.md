---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/elasticsearch.ipynb
---
# Elasticsearch

>[Elasticsearch](https://www.elastic.co/elasticsearch/) is a distributed, RESTful search and analytics engine, capable of performing both vector and lexical search. It is built on top of the Apache Lucene library. 

This notebook shows how to use functionality related to the `Elasticsearch` database.


```python
%pip install --upgrade --quiet langchain-elasticsearch langchain-openai tiktoken langchain
```

## Running and connecting to Elasticsearch

There are two main ways to setup an Elasticsearch instance for use with:

1. Elastic Cloud: Elastic Cloud is a managed Elasticsearch service. Signup for a [free trial](https://cloud.elastic.co/registration?utm_source=langchain&utm_content=documentation).

To connect to an Elasticsearch instance that does not require
login credentials (starting the docker instance with security enabled), pass the Elasticsearch URL and index name along with the
embedding object to the constructor.

2. Local Install Elasticsearch: Get started with Elasticsearch by running it locally. The easiest way is to use the official Elasticsearch Docker image. See the [Elasticsearch Docker documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html) for more information.


### Running Elasticsearch via Docker 
Example: Run a single-node Elasticsearch instance with security disabled. This is not recommended for production use.

```bash
docker run -p 9200:9200 -e "discovery.type=single-node" -e "xpack.security.enabled=false" -e "xpack.security.http.ssl.enabled=false" docker.elastic.co/elasticsearch/elasticsearch:8.12.1
```

Once the Elasticsearch instance is running, you can connect to it using the Elasticsearch URL and index name along with the embedding object to the constructor.

Example:
```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings

embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    es_url="http://localhost:9200",
    index_name="test_index",
    embedding=embedding
)
```
### Authentication
For production, we recommend you run with security enabled. To connect with login credentials, you can use the parameters `es_api_key` or `es_user` and `es_password`.

Example:
```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings

embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    es_url="http://localhost:9200",
    index_name="test_index",
    embedding=embedding,
    es_user="elastic",
    es_password="changeme"
)
```

You can also use an `Elasticsearch` client object that gives you more flexibility, for example to configure the maximum number of retries.

Example:
```python
import elasticsearch
from langchain_elasticsearch import ElasticsearchStore

es_client= elasticsearch.Elasticsearch(
    hosts=["http://localhost:9200"],
    es_user="elastic",
    es_password="changeme"
    max_retries=10,
)

embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    index_name="test_index",
    es_connection=es_client,
    embedding=embedding,
)
```

#### How to obtain a password for the default "elastic" user?

To obtain your Elastic Cloud password for the default "elastic" user:
1. Log in to the Elastic Cloud console at https://cloud.elastic.co
2. Go to "Security" > "Users"
3. Locate the "elastic" user and click "Edit"
4. Click "Reset password"
5. Follow the prompts to reset the password

#### How to obtain an API key?

To obtain an API key:
1. Log in to the Elastic Cloud console at https://cloud.elastic.co
2. Open Kibana and go to Stack Management > API Keys
3. Click "Create API key"
4. Enter a name for the API key and click "Create"
5. Copy the API key and paste it into the `api_key` parameter

### Elastic Cloud
To connect to an Elasticsearch instance on Elastic Cloud, you can use either the `es_cloud_id` parameter or `es_url`.

Example:
```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings

embedding = OpenAIEmbeddings()
elastic_vector_search = ElasticsearchStore(
    es_cloud_id="<cloud_id>",
    index_name="test_index",
    embedding=embedding,
    es_user="elastic",
    es_password="changeme"
)
```

To use the `OpenAIEmbeddings` we have to configure the OpenAI API Key in the environment.


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

## Basic Example
This example we are going to load "state_of_the_union.txt" via the TextLoader, chunk the text into 500 word chunks, and then index each chunk into Elasticsearch.

Once the data is indexed, we perform a simple query to find the top 4 chunks that similar to the query "What did the president say about Ketanji Brown Jackson".

Elasticsearch is running locally on localhost:9200 with [docker](#running-elasticsearch-via-docker). For more details on how to connect to Elasticsearch from Elastic Cloud, see [connecting with authentication](#authentication) above.



```python
from langchain_elasticsearch import ElasticsearchStore
from langchain_openai import OpenAIEmbeddings
```


```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```


```python
db = ElasticsearchStore.from_documents(
    docs,
    embeddings,
    es_url="http://localhost:9200",
    index_name="test-basic",
)

db.client.indices.refresh(index="test-basic")

query = "What did the president say about Ketanji Brown Jackson"
results = db.similarity_search(query)
print(results)
```
```output
[Document(page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../how_to/state_of_the_union.txt'}), Document(page_content='As I said last year, especially to our younger transgender Americans, I will always have your back as your President, so you can be yourself and reach your God-given potential. \n\nWhile it often appears that we never agree, that isn’t true. I signed 80 bipartisan bills into law last year. From preventing government shutdowns to protecting Asian-Americans from still-too-common hate crimes to reforming military justice.', metadata={'source': '../../how_to/state_of_the_union.txt'}), Document(page_content='A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. \n\nAnd if we are to advance liberty and justice, we need to secure the Border and fix the immigration system.', metadata={'source': '../../how_to/state_of_the_union.txt'}), Document(page_content='This is personal to me and Jill, to Kamala, and to so many of you. \n\nCancer is the #2 cause of death in America–second only to heart disease. \n\nLast month, I announced our plan to supercharge  \nthe Cancer Moonshot that President Obama asked me to lead six years ago. \n\nOur goal is to cut the cancer death rate by at least 50% over the next 25 years, turn more cancers from death sentences into treatable diseases.  \n\nMore support for patients and families.', metadata={'source': '../../how_to/state_of_the_union.txt'})]
```
# Metadata

`ElasticsearchStore` supports metadata to stored along with the document. This metadata dict object is stored in a metadata object field in the Elasticsearch document. Based on the metadata value, Elasticsearch will automatically setup the mapping by infering the data type of the metadata value. For example, if the metadata value is a string, Elasticsearch will setup the mapping for the metadata object field as a string type.


```python
# Adding metadata to documents
for i, doc in enumerate(docs):
    doc.metadata["date"] = f"{range(2010, 2020)[i % 10]}-01-01"
    doc.metadata["rating"] = range(1, 6)[i % 5]
    doc.metadata["author"] = ["John Doe", "Jane Doe"][i % 2]

db = ElasticsearchStore.from_documents(
    docs, embeddings, es_url="http://localhost:9200", index_name="test-metadata"
)

query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
print(docs[0].metadata)
```
```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2016-01-01', 'rating': 2, 'author': 'John Doe'}
```
## Filtering Metadata
With metadata added to the documents, you can add metadata filtering at query time. 

### Example: Filter by Exact keyword
Notice: We are using the keyword subfield thats not analyzed


```python
docs = db.similarity_search(
    query, filter=[{"term": {"metadata.author.keyword": "John Doe"}}]
)
print(docs[0].metadata)
```
```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2016-01-01', 'rating': 2, 'author': 'John Doe'}
```
### Example: Filter by Partial Match
This example shows how to filter by partial match. This is useful when you don't know the exact value of the metadata field. For example, if you want to filter by the metadata field `author` and you don't know the exact value of the author, you can use a partial match to filter by the author's last name. Fuzzy matching is also supported.

"Jon" matches on "John Doe" as "Jon" is a close match to "John" token.


```python
docs = db.similarity_search(
    query,
    filter=[{"match": {"metadata.author": {"query": "Jon", "fuzziness": "AUTO"}}}],
)
print(docs[0].metadata)
```
```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2016-01-01', 'rating': 2, 'author': 'John Doe'}
```
### Example: Filter by Date Range


```python
docs = db.similarity_search(
    "Any mention about Fred?",
    filter=[{"range": {"metadata.date": {"gte": "2010-01-01"}}}],
)
print(docs[0].metadata)
```
```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2012-01-01', 'rating': 3, 'author': 'John Doe', 'geo_location': {'lat': 40.12, 'lon': -71.34}}
```
### Example: Filter by Numeric Range


```python
docs = db.similarity_search(
    "Any mention about Fred?", filter=[{"range": {"metadata.rating": {"gte": 2}}}]
)
print(docs[0].metadata)
```
```output
{'source': '../../how_to/state_of_the_union.txt', 'date': '2012-01-01', 'rating': 3, 'author': 'John Doe', 'geo_location': {'lat': 40.12, 'lon': -71.34}}
```
### Example: Filter by Geo Distance
Requires an index with a geo_point mapping to be declared for `metadata.geo_location`.


```python
docs = db.similarity_search(
    "Any mention about Fred?",
    filter=[
        {
            "geo_distance": {
                "distance": "200km",
                "metadata.geo_location": {"lat": 40, "lon": -70},
            }
        }
    ],
)
print(docs[0].metadata)
```

Filter supports many more types of queries than above. 

Read more about them in the [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html).

# Distance Similarity Algorithm
Elasticsearch supports the following vector distance similarity algorithms:

- cosine
- euclidean
- dot_product

The cosine similarity algorithm is the default.

You can specify the similarity Algorithm needed via the similarity parameter.

**NOTE**
Depending on the retrieval strategy, the similarity algorithm cannot be changed at query time. It is needed to be set when creating the index mapping for field. If you need to change the similarity algorithm, you need to delete the index and recreate it with the correct distance_strategy.

```python

db = ElasticsearchStore.from_documents(
    docs, 
    embeddings, 
    es_url="http://localhost:9200", 
    index_name="test",
    distance_strategy="COSINE"
    # distance_strategy="EUCLIDEAN_DISTANCE"
    # distance_strategy="DOT_PRODUCT"
)

```

# Retrieval Strategies
Elasticsearch has big advantages over other vector only databases from its ability to support a wide range of retrieval strategies. In this notebook we will configure `ElasticsearchStore` to support some of the most common retrieval strategies. 

By default, `ElasticsearchStore` uses the `DenseVectorStrategy` (was called `ApproxRetrievalStrategy` prior to version 0.2.0).

## DenseVectorStrategy
This will return the top `k` most similar vectors to the query vector.  The `k` parameter is set when the `ElasticsearchStore` is initialized. The default value is `10`.


```python
from langchain_elasticsearch import DenseVectorStrategy

db = ElasticsearchStore.from_documents(
    docs,
    embeddings,
    es_url="http://localhost:9200",
    index_name="test",
    strategy=DenseVectorStrategy(),
)

docs = db.similarity_search(
    query="What did the president say about Ketanji Brown Jackson?", k=10
)
```

### Example: Hybrid retrieval with dense vector and keyword search
This example will show how to configure `ElasticsearchStore` to perform a hybrid retrieval, using a combination of approximate semantic search and keyword based search. 

We use RRF to balance the two scores from different retrieval methods.

To enable hybrid retrieval, we need to set `hybrid=True` in the `DenseVectorStrategy` constructor.

```python

db = ElasticsearchStore.from_documents(
    docs, 
    embeddings, 
    es_url="http://localhost:9200", 
    index_name="test",
    strategy=DenseVectorStrategy(hybrid=True)
)
```

When `hybrid` is enabled, the query performed will be a combination of approximate semantic search and keyword based search. 

It will use `rrf` (Reciprocal Rank Fusion) to balance the two scores from different retrieval methods.

**Note** RRF requires Elasticsearch 8.9.0 or above.

```json
{
    "knn": {
        "field": "vector",
        "filter": [],
        "k": 1,
        "num_candidates": 50,
        "query_vector": [1.0, ..., 0.0],
    },
    "query": {
        "bool": {
            "filter": [],
            "must": [{"match": {"text": {"query": "foo"}}}],
        }
    },
    "rank": {"rrf": {}},
}
```

### Example: Dense vector search with Embedding Model in Elasticsearch
This example will show how to configure `ElasticsearchStore` to use the embedding model deployed in Elasticsearch for dense vector retrieval.

To use this, specify the model_id in `DenseVectorStrategy` constructor via the `query_model_id` argument.

**NOTE** This requires the model to be deployed and running in Elasticsearch ml node. See [notebook example](https://github.com/elastic/elasticsearch-labs/blob/main/notebooks/integrations/hugging-face/loading-model-from-hugging-face.ipynb) on how to deploy the model with eland.



```python
DENSE_SELF_DEPLOYED_INDEX_NAME = "test-dense-self-deployed"

# Note: This does not have an embedding function specified
# Instead, we will use the embedding model deployed in Elasticsearch
db = ElasticsearchStore(
    es_cloud_id="<your cloud id>",
    es_user="elastic",
    es_password="<your password>",
    index_name=DENSE_SELF_DEPLOYED_INDEX_NAME,
    query_field="text_field",
    vector_query_field="vector_query_field.predicted_value",
    strategy=DenseVectorStrategy(model_id="sentence-transformers__all-minilm-l6-v2"),
)

# Setup a Ingest Pipeline to perform the embedding
# of the text field
db.client.ingest.put_pipeline(
    id="test_pipeline",
    processors=[
        {
            "inference": {
                "model_id": "sentence-transformers__all-minilm-l6-v2",
                "field_map": {"query_field": "text_field"},
                "target_field": "vector_query_field",
            }
        }
    ],
)

# creating a new index with the pipeline,
# not relying on langchain to create the index
db.client.indices.create(
    index=DENSE_SELF_DEPLOYED_INDEX_NAME,
    mappings={
        "properties": {
            "text_field": {"type": "text"},
            "vector_query_field": {
                "properties": {
                    "predicted_value": {
                        "type": "dense_vector",
                        "dims": 384,
                        "index": True,
                        "similarity": "l2_norm",
                    }
                }
            },
        }
    },
    settings={"index": {"default_pipeline": "test_pipeline"}},
)

db.from_texts(
    ["hello world"],
    es_cloud_id="<cloud id>",
    es_user="elastic",
    es_password="<cloud password>",
    index_name=DENSE_SELF_DEPLOYED_INDEX_NAME,
    query_field="text_field",
    vector_query_field="vector_query_field.predicted_value",
    strategy=DenseVectorStrategy(model_id="sentence-transformers__all-minilm-l6-v2"),
)

# Perform search
db.similarity_search("hello world", k=10)
```

## SparseVectorStrategy (ELSER)
This strategy uses Elasticsearch's sparse vector retrieval to retrieve the top-k results. We only support our own "ELSER" embedding model for now.

**NOTE** This requires the ELSER model to be deployed and running in Elasticsearch ml node. 

To use this, specify `SparseVectorStrategy` (was called `SparseVectorRetrievalStrategy` prior to version 0.2.0) in the `ElasticsearchStore` constructor. You will need to provide a model ID.


```python
from langchain_elasticsearch import SparseVectorStrategy

# Note that this example doesn't have an embedding function. This is because we infer the tokens at index time and at query time within Elasticsearch.
# This requires the ELSER model to be loaded and running in Elasticsearch.
db = ElasticsearchStore.from_documents(
    docs,
    es_cloud_id="<cloud id>",
    es_user="elastic",
    es_password="<cloud password>",
    index_name="test-elser",
    strategy=SparseVectorStrategy(model_id=".elser_model_2"),
)

db.client.indices.refresh(index="test-elser")

results = db.similarity_search(
    "What did the president say about Ketanji Brown Jackson", k=4
)
print(results[0])
```
```output
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
```
## DenseVectorScriptScoreStrategy
This strategy uses Elasticsearch's script score query to perform exact vector retrieval (also known as brute force) to retrieve the top-k results. (This strategy was called `ExactRetrievalStrategy` prior to version 0.2.0.)

To use this, specify `DenseVectorScriptScoreStrategy` in `ElasticsearchStore` constructor.

```python
from langchain_elasticsearch import SparseVectorStrategy

db = ElasticsearchStore.from_documents(
    docs, 
    embeddings, 
    es_url="http://localhost:9200", 
    index_name="test",
    strategy=DenseVectorScriptScoreStrategy(),
)
```

## BM25Strategy
Finally, you can use full-text keyword search.

To use this, specify `BM25Strategy` in `ElasticsearchStore` constructor.

```python
from langchain_elasticsearch import BM25Strategy

db = ElasticsearchStore.from_documents(
    docs, 
    es_url="http://localhost:9200", 
    index_name="test",
    strategy=BM25Strategy(),
)
```

## BM25RetrievalStrategy
This strategy allows the user to perform searches using pure BM25 without vector search.

To use this, specify `BM25RetrievalStrategy` in `ElasticsearchStore` constructor.

Note that in the example below, the embedding option is not specified, indicating that the search is conducted without using embeddings.


```python
from langchain_elasticsearch import ElasticsearchStore

db = ElasticsearchStore(
    es_url="http://localhost:9200",
    index_name="test_index",
    strategy=ElasticsearchStore.BM25RetrievalStrategy(),
)

db.add_texts(
    ["foo", "foo bar", "foo bar baz", "bar", "bar baz", "baz"],
)

results = db.similarity_search(query="foo", k=10)
print(results)
```
```output
[Document(page_content='foo'), Document(page_content='foo bar'), Document(page_content='foo bar baz')]
```
## Customise the Query
With `custom_query` parameter at search, you are able to adjust the query that is used to retrieve documents from Elasticsearch. This is useful if you want to use a more complex query, to support linear boosting of fields.


```python
# Example of a custom query thats just doing a BM25 search on the text field.
def custom_query(query_body: dict, query: str):
    """Custom query to be used in Elasticsearch.
    Args:
        query_body (dict): Elasticsearch query body.
        query (str): Query string.
    Returns:
        dict: Elasticsearch query body.
    """
    print("Query Retriever created by the retrieval strategy:")
    print(query_body)
    print()

    new_query_body = {"query": {"match": {"text": query}}}

    print("Query thats actually used in Elasticsearch:")
    print(new_query_body)
    print()

    return new_query_body


results = db.similarity_search(
    "What did the president say about Ketanji Brown Jackson",
    k=4,
    custom_query=custom_query,
)
print("Results:")
print(results[0])
```
```output
Query Retriever created by the retrieval strategy:
{'query': {'bool': {'must': [{'text_expansion': {'vector.tokens': {'model_id': '.elser_model_1', 'model_text': 'What did the president say about Ketanji Brown Jackson'}}}], 'filter': []}}}

Query thats actually used in Elasticsearch:
{'query': {'match': {'text': 'What did the president say about Ketanji Brown Jackson'}}}

Results:
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
```
# Customize the Document Builder

With ```doc_builder``` parameter at search, you are able to adjust how a Document is being built using data retrieved from Elasticsearch. This is especially useful if you have indices which were not created using Langchain.


```python
from typing import Dict

from langchain_core.documents import Document


def custom_document_builder(hit: Dict) -> Document:
    src = hit.get("_source", {})
    return Document(
        page_content=src.get("content", "Missing content!"),
        metadata={
            "page_number": src.get("page_number", -1),
            "original_filename": src.get("original_filename", "Missing filename!"),
        },
    )


results = db.similarity_search(
    "What did the president say about Ketanji Brown Jackson",
    k=4,
    doc_builder=custom_document_builder,
)
print("Results:")
print(results[0])
```

# FAQ

## Question: Im getting timeout errors when indexing documents into Elasticsearch. How do I fix this?
One possible issue is your documents might take longer to index into Elasticsearch. ElasticsearchStore uses the Elasticsearch bulk API which has a few defaults that you can adjust to reduce the chance of timeout errors.

This is also a good idea when you're using SparseVectorRetrievalStrategy.

The defaults are:
- `chunk_size`: 500
- `max_chunk_bytes`: 100MB

To adjust these, you can pass in the `chunk_size` and `max_chunk_bytes` parameters to the ElasticsearchStore `add_texts` method.

```python
    vector_store.add_texts(
        texts,
        bulk_kwargs={
            "chunk_size": 50,
            "max_chunk_bytes": 200000000
        }
    )
```

# Upgrading to ElasticsearchStore

If you're already using Elasticsearch in your langchain based project, you may be using the old implementations: `ElasticVectorSearch` and `ElasticKNNSearch` which are now deprecated. We've introduced a new implementation called `ElasticsearchStore` which is more flexible and easier to use. This notebook will guide you through the process of upgrading to the new implementation.

## What's new?

The new implementation is now one class called `ElasticsearchStore` which can be used for approximate dense vector, exact dense vector, sparse vector (ELSER), BM25 retrieval and hybrid retrieval, via strategies.

## I am using ElasticKNNSearch

Old implementation:

```python

from langchain_community.vectorstores.elastic_vector_search import ElasticKNNSearch

db = ElasticKNNSearch(
  elasticsearch_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding
)

```

New implementation:

```python

from langchain_elasticsearch import ElasticsearchStore, DenseVectorStrategy

db = ElasticsearchStore(
  es_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding,
  # if you use the model_id
  # strategy=DenseVectorStrategy(model_id="test_model")
  # if you use hybrid search
  # strategy=DenseVectorStrategy(hybrid=True)
)

```

## I am using ElasticVectorSearch

Old implementation:

```python

from langchain_community.vectorstores.elastic_vector_search import ElasticVectorSearch

db = ElasticVectorSearch(
  elasticsearch_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding
)

```

New implementation:

```python

from langchain_elasticsearch import ElasticsearchStore, DenseVectorScriptScoreStrategy

db = ElasticsearchStore(
  es_url="http://localhost:9200",
  index_name="test_index",
  embedding=embedding,
  strategy=DenseVectorScriptScoreStrategy()
)

```


```python
db.client.indices.delete(
    index="test-metadata, test-elser, test-basic",
    ignore_unavailable=True,
    allow_no_indices=True,
)
```



```output
ObjectApiResponse({'acknowledged': True})
```



## Related

- Vector store [conceptual guide](/docs/concepts/#vector-stores)
- Vector store [how-to guides](/docs/how_to/#vector-stores)
