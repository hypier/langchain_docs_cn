---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/couchbase.ipynb
---
# Couchbase 
[Couchbase](http://couchbase.com/) is an award-winning distributed NoSQL cloud database that delivers unmatched versatility, performance, scalability, and financial value for all of your cloud, mobile, AI, and edge computing applications. Couchbase embraces AI with coding assistance for developers and vector search for their applications.

Vector Search is a part of the [Full Text Search Service](https://docs.couchbase.com/server/current/learn/services-and-indexes/services/search-service.html) (Search Service) in Couchbase.

This tutorial explains how to use Vector Search in Couchbase. You can work with both [Couchbase Capella](https://www.couchbase.com/products/capella/) and your self-managed Couchbase Server.

## Installation


```python
%pip install --upgrade --quiet langchain langchain-openai langchain-couchbase
```


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

## Import the Vector Store and Embeddings


```python
from langchain_couchbase.vectorstores import CouchbaseVectorStore
from langchain_openai import OpenAIEmbeddings
```

## Create Couchbase Connection Object
We create a connection to the Couchbase cluster initially and then pass the cluster object to the Vector Store. 

Here, we are connecting using the username and password. You can also connect using any other supported way to your cluster. 

For more information on connecting to the Couchbase cluster, please check the [Python SDK documentation](https://docs.couchbase.com/python-sdk/current/hello-world/start-using-sdk.html#connect).


```python
COUCHBASE_CONNECTION_STRING = (
    "couchbase://localhost"  # or "couchbases://localhost" if using TLS
)
DB_USERNAME = "Administrator"
DB_PASSWORD = "Password"
```


```python
from datetime import timedelta

from couchbase.auth import PasswordAuthenticator
from couchbase.cluster import Cluster
from couchbase.options import ClusterOptions

auth = PasswordAuthenticator(DB_USERNAME, DB_PASSWORD)
options = ClusterOptions(auth)
cluster = Cluster(COUCHBASE_CONNECTION_STRING, options)

# Wait until the cluster is ready for use.
cluster.wait_until_ready(timedelta(seconds=5))
```

We will now set the bucket, scope, and collection names in the Couchbase cluster that we want to use for Vector Search. 

For this example, we are using the default scope & collections.


```python
BUCKET_NAME = "testing"
SCOPE_NAME = "_default"
COLLECTION_NAME = "_default"
SEARCH_INDEX_NAME = "vector-index"
```

For this tutorial, we will use OpenAI embeddings


```python
embeddings = OpenAIEmbeddings()
```

## Create the Search Index
Currently, the Search index needs to be created from the Couchbase Capella or Server UI or using the REST interface. 

Let us define a Search index with the name `vector-index` on the testing bucket

For this example, let us use the Import Index feature on the Search Service on the UI. 

We are defining an index on the `testing` bucket's `_default` scope on the `_default` collection with the vector field set to `embedding` with 1536 dimensions and the text field set to `text`. We are also indexing and storing all the fields under `metadata` in the document as a dynamic mapping to account for varying document structures. The similarity metric is set to `dot_product`.

### How to Import an Index to the Full Text Search service?
 - [Couchbase Server](https://docs.couchbase.com/server/current/search/import-search-index.html)
     - Click on Search -> Add Index -> Import
     - Copy the following Index definition in the Import screen
     - Click on Create Index to create the index.
 - [Couchbase Capella](https://docs.couchbase.com/cloud/search/import-search-index.html)
     - Copy the index definition to a new file `index.json`
     - Import the file in Capella using the instructions in the documentation.
     - Click on Create Index to create the index.
  


### Index Definition
```
{
 "name": "vector-index",
 "type": "fulltext-index",
 "params": {
  "doc_config": {
   "docid_prefix_delim": "",
   "docid_regexp": "",
   "mode": "type_field",
   "type_field": "type"
  },
  "mapping": {
   "default_analyzer": "standard",
   "default_datetime_parser": "dateTimeOptional",
   "default_field": "_all",
   "default_mapping": {
    "dynamic": true,
    "enabled": true,
    "properties": {
     "metadata": {
      "dynamic": true,
      "enabled": true
     },
     "embedding": {
      "enabled": true,
      "dynamic": false,
      "fields": [
       {
        "dims": 1536,
        "index": true,
        "name": "embedding",
        "similarity": "dot_product",
        "type": "vector",
        "vector_index_optimized_for": "recall"
       }
      ]
     },
     "text": {
      "enabled": true,
      "dynamic": false,
      "fields": [
       {
        "index": true,
        "name": "text",
        "store": true,
        "type": "text"
       }
      ]
     }
    }
   },
   "default_type": "_default",
   "docvalues_dynamic": false,
   "index_dynamic": true,
   "store_dynamic": true,
   "type_field": "_type"
  },
  "store": {
   "indexType": "scorch",
   "segmentVersion": 16
  }
 },
 "sourceType": "gocbcore",
 "sourceName": "testing",
 "sourceParams": {},
 "planParams": {
  "maxPartitionsPerPIndex": 103,
  "indexPartitions": 10,
  "numReplicas": 0
 }
}
```

For more details on how to create a Search index with support for Vector fields, please refer to the documentation.

- [Couchbase Capella](https://docs.couchbase.com/cloud/vector-search/create-vector-search-index-ui.html)
  
- [Couchbase Server](https://docs.couchbase.com/server/current/vector-search/create-vector-search-index-ui.html)

## Create Vector Store
We create the vector store object with the cluster information and the search index name.


```python
vector_store = CouchbaseVectorStore(
    cluster=cluster,
    bucket_name=BUCKET_NAME,
    scope_name=SCOPE_NAME,
    collection_name=COLLECTION_NAME,
    embedding=embeddings,
    index_name=SEARCH_INDEX_NAME,
)
```

### Specify the Text & Embeddings Field
You can optionally specify the text & embeddings field for the document using the `text_key` and `embedding_key` fields.
```
vector_store = CouchbaseVectorStore(
    cluster=cluster,
    bucket_name=BUCKET_NAME,
    scope_name=SCOPE_NAME,
    collection_name=COLLECTION_NAME,
    embedding=embeddings,
    index_name=SEARCH_INDEX_NAME,
    text_key="text",
    embedding_key="embedding",
)
```

## Basic Vector Search Example
For this example, we are going to load the "state_of_the_union.txt" file via the TextLoader, chunk the text into 500 character chunks with no overlaps and index all these chunks into Couchbase.

After the data is indexed, we perform a simple query to find the top 4 chunks that are similar to the query "What did president say about Ketanji Brown Jackson".



```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
```


```python
vector_store = CouchbaseVectorStore.from_documents(
    documents=docs,
    embedding=embeddings,
    cluster=cluster,
    bucket_name=BUCKET_NAME,
    scope_name=SCOPE_NAME,
    collection_name=COLLECTION_NAME,
    index_name=SEARCH_INDEX_NAME,
)
```


```python
query = "What did president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(query)
print(results[0])
```
```output
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
```
## Similarity Search with Score
You can fetch the scores for the results by calling the `similarity_search_with_score` method.


```python
query = "What did president say about Ketanji Brown Jackson"
results = vector_store.similarity_search_with_score(query)
document, score = results[0]
print(document)
print(f"Score: {score}")
```
```output
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
Score: 0.8211871385574341
```
## Specifying Fields to Return
You can specify the fields to return from the document using `fields` parameter in the searches. These fields are returned as part of the `metadata` object in the returned Document. You can fetch any field that is stored in the Search index. The `text_key` of the document is returned as part of the document's `page_content`.

If you do not specify any fields to be fetched, all the fields stored in the index are returned.

If you want to fetch one of the fields in the metadata, you need to specify it using `.`

For example, to fetch the `source` field in the metadata, you need to specify `metadata.source`.



```python
query = "What did president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(query, fields=["metadata.source"])
print(results[0])
```
```output
page_content='One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.' metadata={'source': '../../how_to/state_of_the_union.txt'}
```
## Hybrid Search
Couchbase allows you to do hybrid searches by combining Vector Search results with searches on non-vector fields of the document like the `metadata` object. 

The results will be based on the combination of the results from both Vector Search and the searches supported by Search Service. The scores of each of the component searches are added up to get the total score of the result.

To perform hybrid searches, there is an optional parameter, `search_options` that can be passed to all the similarity searches.  
The different search/query possibilities for the `search_options` can be found [here](https://docs.couchbase.com/server/current/search/search-request-params.html#query-object).

### Create Diverse Metadata for Hybrid Search
In order to simulate hybrid search, let us create some random metadata from the existing documents. 
We uniformly add three fields to the metadata, `date` between 2010 & 2020, `rating` between 1 & 5 and `author` set to either John Doe or Jane Doe. 


```python
# Adding metadata to documents
for i, doc in enumerate(docs):
    doc.metadata["date"] = f"{range(2010, 2020)[i % 10]}-01-01"
    doc.metadata["rating"] = range(1, 6)[i % 5]
    doc.metadata["author"] = ["John Doe", "Jane Doe"][i % 2]

vector_store.add_documents(docs)

query = "What did the president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(query)
print(results[0].metadata)
```
```output
{'author': 'John Doe', 'date': '2016-01-01', 'rating': 2, 'source': '../../how_to/state_of_the_union.txt'}
```
### Example: Search by Exact Value
We can search for exact matches on a textual field like the author in the `metadata` object.


```python
query = "What did the president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(
    query,
    search_options={"query": {"field": "metadata.author", "match": "John Doe"}},
    fields=["metadata.author"],
)
print(results[0])
```
```output
page_content='This is personal to me and Jill, to Kamala, and to so many of you. \n\nCancer is the #2 cause of death in America–second only to heart disease. \n\nLast month, I announced our plan to supercharge  \nthe Cancer Moonshot that President Obama asked me to lead six years ago. \n\nOur goal is to cut the cancer death rate by at least 50% over the next 25 years, turn more cancers from death sentences into treatable diseases.  \n\nMore support for patients and families.' metadata={'author': 'John Doe'}
```
### Example: Search by Partial Match
We can search for partial matches by specifying a fuzziness for the search. This is useful when you want to search for slight variations or misspellings of a search query.

Here, "Jae" is close (fuzziness of 1) to "Jane".


```python
query = "What did the president say about Ketanji Brown Jackson"
results = vector_store.similarity_search(
    query,
    search_options={
        "query": {"field": "metadata.author", "match": "Jae", "fuzziness": 1}
    },
    fields=["metadata.author"],
)
print(results[0])
```
```output
page_content='A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. \n\nAnd if we are to advance liberty and justice, we need to secure the Border and fix the immigration system.' metadata={'author': 'Jane Doe'}
```
### Example: Search by Date Range Query
We can search for documents that are within a date range query on a date field like `metadata.date`.


```python
query = "Any mention about independence?"
results = vector_store.similarity_search(
    query,
    search_options={
        "query": {
            "start": "2016-12-31",
            "end": "2017-01-02",
            "inclusive_start": True,
            "inclusive_end": False,
            "field": "metadata.date",
        }
    },
)
print(results[0])
```
```output
page_content='He will never extinguish their love of freedom. He will never weaken the resolve of the free world. \n\nWe meet tonight in an America that has lived through two of the hardest years this nation has ever faced. \n\nThe pandemic has been punishing. \n\nAnd so many families are living paycheck to paycheck, struggling to keep up with the rising cost of food, gas, housing, and so much more. \n\nI understand.' metadata={'author': 'Jane Doe', 'date': '2017-01-01', 'rating': 3, 'source': '../../how_to/state_of_the_union.txt'}
```
### Example: Search by Numeric Range Query
We can search for documents that are within a range for a numeric field like `metadata.rating`.


```python
query = "Any mention about independence?"
results = vector_store.similarity_search_with_score(
    query,
    search_options={
        "query": {
            "min": 3,
            "max": 5,
            "inclusive_min": True,
            "inclusive_max": True,
            "field": "metadata.rating",
        }
    },
)
print(results[0])
```
```output
(Document(page_content='He will never extinguish their love of freedom. He will never weaken the resolve of the free world. \n\nWe meet tonight in an America that has lived through two of the hardest years this nation has ever faced. \n\nThe pandemic has been punishing. \n\nAnd so many families are living paycheck to paycheck, struggling to keep up with the rising cost of food, gas, housing, and so much more. \n\nI understand.', metadata={'author': 'Jane Doe', 'date': '2017-01-01', 'rating': 3, 'source': '../../how_to/state_of_the_union.txt'}), 0.9000703597577832)
```
### Example: Combining Multiple Search Queries
Different search queries can be combined using AND (conjuncts) or OR (disjuncts) operators.

In this example, we are checking for documents with a rating between 3 & 4 and dated between 2015 & 2018.


```python
query = "Any mention about independence?"
results = vector_store.similarity_search_with_score(
    query,
    search_options={
        "query": {
            "conjuncts": [
                {"min": 3, "max": 4, "inclusive_max": True, "field": "metadata.rating"},
                {"start": "2016-12-31", "end": "2017-01-02", "field": "metadata.date"},
            ]
        }
    },
)
print(results[0])
```
```output
(Document(page_content='He will never extinguish their love of freedom. He will never weaken the resolve of the free world. \n\nWe meet tonight in an America that has lived through two of the hardest years this nation has ever faced. \n\nThe pandemic has been punishing. \n\nAnd so many families are living paycheck to paycheck, struggling to keep up with the rising cost of food, gas, housing, and so much more. \n\nI understand.', metadata={'author': 'Jane Doe', 'date': '2017-01-01', 'rating': 3, 'source': '../../how_to/state_of_the_union.txt'}), 1.3598770370389914)
```
### Other Queries
Similarly, you can use any of the supported Query methods like Geo Distance, Polygon Search, Wildcard, Regular Expressions, etc in the `search_options` parameter. Please refer to the documentation for more details on the available query methods and their syntax.

- [Couchbase Capella](https://docs.couchbase.com/cloud/search/search-request-params.html#query-object)
- [Couchbase Server](https://docs.couchbase.com/server/current/search/search-request-params.html#query-object)

# Frequently Asked Questions

## Question: Should I create the Search index before creating the CouchbaseVectorStore object?
Yes, currently you need to create the Search index before creating the `CouchbaseVectoreStore` object.


## Question: I am not seeing all the fields that I specified in my search results. 

In Couchbase, we can only return the fields stored in the Search index. Please ensure that the field that you are trying to access in the search results is part of the Search index.

One way to handle this is to index and store a document's fields dynamically in the index. 

- In Capella, you need to go to "Advanced Mode" then under the chevron "General Settings" you can check "[X] Store Dynamic Fields" or "[X] Index Dynamic Fields"
- In Couchbase Server, in the Index Editor (not Quick Editor) under the chevron  "Advanced" you can check "[X] Store Dynamic Fields" or "[X] Index Dynamic Fields"

Note that these options will increase the size of the index.

For more details on dynamic mappings, please refer to the [documentation](https://docs.couchbase.com/cloud/search/customize-index.html).


## Question: I am unable to see the metadata object in my search results. 
This is most likely due to the `metadata` field in the document not being indexed and/or stored by the Couchbase Search index. In order to index the `metadata` field in the document, you need to add it to the index as a child mapping. 

If you select to map all the fields in the mapping, you will be able to search by all metadata fields. Alternatively, to optimize the index, you can select the specific fields inside `metadata` object to be indexed. You can refer to the [docs](https://docs.couchbase.com/cloud/search/customize-index.html) to learn more about indexing child mappings.

Creating Child Mappings

* [Couchbase Capella](https://docs.couchbase.com/cloud/search/create-child-mapping.html)
* [Couchbase Server](https://docs.couchbase.com/server/current/search/create-child-mapping.html)


## Related

- Vector store [conceptual guide](/docs/concepts/#vector-stores)
- Vector store [how-to guides](/docs/how_to/#vector-stores)
