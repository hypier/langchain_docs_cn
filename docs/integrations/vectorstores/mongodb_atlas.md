---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/mongodb_atlas.ipynb
---
# MongoDB Atlas

This notebook covers how to MongoDB Atlas vector search in LangChain, using the `langchain-mongodb` package.

>[MongoDB Atlas](https://www.mongodb.com/docs/atlas/) is a fully-managed cloud database available in AWS, Azure, and GCP.  It supports native Vector Search and full text search (BM25) on your MongoDB document data.

>[MongoDB Atlas Vector Search](https://www.mongodb.com/products/platform/atlas-vector-search) allows to store your embeddings in MongoDB documents, create a vector search index, and perform KNN search with an approximate nearest neighbor algorithm (`Hierarchical Navigable Small Worlds`). It uses the [$vectorSearch MQL Stage](https://www.mongodb.com/docs/atlas/atlas-vector-search/vector-search-overview/). 

## Prerequisites
>*An Atlas cluster running MongoDB version 6.0.11, 7.0.2, or later (including RCs).

>*An OpenAI API Key. You must have a paid OpenAI account with credits available for API requests.

You'll need to install `langchain-mongodb` to use this integration

## Setting up MongoDB Atlas Cluster
To use MongoDB Atlas, you must first deploy a cluster. We have a Forever-Free tier of clusters available. To get started head over to Atlas here: [quick start](https://www.mongodb.com/docs/atlas/getting-started/).

## Usage
In the notebook we will demonstrate how to perform `Retrieval Augmented Generation` (RAG) using MongoDB Atlas, OpenAI and Langchain. We will be performing Similarity Search, Similarity Search with Metadata Pre-Filtering, and Question Answering over the PDF document for [GPT 4 technical report](https://arxiv.org/pdf/2303.08774.pdf) that came out in March 2023 and hence is not part of the OpenAI's Large Language Model(LLM)'s parametric memory, which had a knowledge cutoff of September 2021.

We want to use `OpenAIEmbeddings` so we need to set up our OpenAI API Key. 


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

Now we will setup the environment variables for the MongoDB Atlas cluster


```python
%pip install --upgrade --quiet langchain langchain-mongodb pypdf pymongo langchain-openai tiktoken
```


```python
import getpass

MONGODB_ATLAS_CLUSTER_URI = getpass.getpass("MongoDB Atlas Cluster URI:")
```


```python
from pymongo import MongoClient

# initialize MongoDB python client
client = MongoClient(MONGODB_ATLAS_CLUSTER_URI)

DB_NAME = "langchain_db"
COLLECTION_NAME = "test"
ATLAS_VECTOR_SEARCH_INDEX_NAME = "index_name"

MONGODB_COLLECTION = client[DB_NAME][COLLECTION_NAME]
```

## Create Vector Search Index

Now, let's create a vector search index on your cluster. More detailed steps can be found at [Create Vector Search Index for LangChain](https://www.mongodb.com/docs/atlas/atlas-vector-search/ai-integrations/langchain/#create-the-atlas-vector-search-index) section.
In the below example, `embedding` is the name of the field that contains the embedding vector. Please refer to the [documentation](https://www.mongodb.com/docs/atlas/atlas-vector-search/create-index/) to get more details on how to define an Atlas Vector Search index.
You can name the index `{ATLAS_VECTOR_SEARCH_INDEX_NAME}` and create the index on the namespace `{DB_NAME}.{COLLECTION_NAME}`. Finally, write the following definition in the JSON editor on MongoDB Atlas:

```json
{
  "fields":[
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 1536,
      "similarity": "cosine"
    }
  ]
}
```

Additionally, if you are running a MongoDB M10 cluster with server version 6.0+, you can leverage the `MongoDBAtlasVectorSearch.create_index`. To add the above index its usage would look like this.

```python
from langchain_community.embeddings.openai import OpenAIEmbeddings
from langchain_mongodb.vectorstores import MongoDBAtlasVectorSearch
from pymongo import MongoClient

mongo_client = MongoClient("<YOUR-CONNECTION-STRING>")
collection = mongo_client["<db_name>"]["<collection_name>"]
embeddings = OpenAIEmbeddings()

vectorstore = MongoDBAtlasVectorSearch(
  collection=collection,
  embedding=embeddings,
  index_name="<ATLAS_VECTOR_SEARCH_INDEX_NAME>",
  relevance_score_fn="cosine",
)

# Creates an index using the index_name provided and relevance_score_fn type
vectorstore.create_index(dimensions=1536)
```

# Insert Data


```python
from langchain_community.document_loaders import PyPDFLoader

# Load the PDF
loader = PyPDFLoader("https://arxiv.org/pdf/2303.08774.pdf")
data = loader.load()
```


```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=150)
docs = text_splitter.split_documents(data)
```


```python
print(docs[0])
```


```python
from langchain_community.vectorstores import MongoDBAtlasVectorSearch
from langchain_openai import OpenAIEmbeddings

# insert the documents in MongoDB Atlas with their embedding
vector_search = MongoDBAtlasVectorSearch.from_documents(
    documents=docs,
    embedding=OpenAIEmbeddings(disallowed_special=()),
    collection=MONGODB_COLLECTION,
    index_name=ATLAS_VECTOR_SEARCH_INDEX_NAME,
)
```


```python
# Perform a similarity search between the embedding of the query and the embeddings of the documents
query = "What were the compute requirements for training GPT 4"
results = vector_search.similarity_search(query)

print(results[0].page_content)
```

# Querying data

We can also instantiate the vector store directly and execute a query as follows:


```python
from langchain_community.vectorstores import MongoDBAtlasVectorSearch
from langchain_openai import OpenAIEmbeddings

vector_search = MongoDBAtlasVectorSearch.from_connection_string(
    MONGODB_ATLAS_CLUSTER_URI,
    DB_NAME + "." + COLLECTION_NAME,
    OpenAIEmbeddings(disallowed_special=()),
    index_name=ATLAS_VECTOR_SEARCH_INDEX_NAME,
)
```

## Pre-filtering with Similarity Search

Atlas Vector Search supports pre-filtering using MQL Operators for filtering.  Below is an example index and query on the same data loaded above that allows you do metadata filtering on the "page" field.  You can update your existing index with the filter defined and do pre-filtering with vector search.

```json
{
  "fields":[
    {
      "type": "vector",
      "path": "embedding",
      "numDimensions": 1536,
      "similarity": "cosine"
    },
    {
      "type": "filter",
      "path": "page"
    }
  ]
}
```

You can also update the index programmatically using the `MongoDBAtlasVectorSearch.create_index` method.

```python
vectorstore.create_index(
  dimensions=1536,
  filters=[{"type":"filter", "path":"page"}],
  update=True
)
```


```python
query = "What were the compute requirements for training GPT 4"

results = vector_search.similarity_search_with_score(
    query=query, k=5, pre_filter={"page": {"$eq": 1}}
)

# Display results
for result in results:
    print(result)
```

## Similarity Search with Score


```python
query = "What were the compute requirements for training GPT 4"

results = vector_search.similarity_search_with_score(
    query=query,
    k=5,
)

# Display results
for result in results:
    print(result)
```

## Question Answering 


```python
qa_retriever = vector_search.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 25},
)
```


```python
from langchain_core.prompts import PromptTemplate

prompt_template = """Use the following pieces of context to answer the question at the end. If you don't know the answer, just say that you don't know, don't try to make up an answer.

{context}

Question: {question}
"""
PROMPT = PromptTemplate(
    template=prompt_template, input_variables=["context", "question"]
)
```


```python
from langchain.chains import RetrievalQA
from langchain_openai import OpenAI

qa = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",
    retriever=qa_retriever,
    return_source_documents=True,
    chain_type_kwargs={"prompt": PROMPT},
)

docs = qa({"query": "gpt-4 compute requirements"})

print(docs["result"])
print(docs["source_documents"])
```

GPT-4 requires significantly more compute than earlier GPT models. On a dataset derived from OpenAI's internal codebase, GPT-4 requires 100p (petaflops) of compute to reach the lowest loss, while the smaller models require 1-10n (nanoflops).

# Other Notes
>* More documentation can be found at [LangChain-MongoDB](https://www.mongodb.com/docs/atlas/atlas-vector-search/ai-integrations/langchain/) site
>* This feature is Generally Available and ready for production deployments.
>* The langchain version 0.0.305 ([release notes](https://github.com/langchain-ai/langchain/releases/tag/v0.0.305)) introduces the support for $vectorSearch MQL stage, which is available with MongoDB Atlas 6.0.11 and 7.0.2. Users utilizing earlier versions of MongoDB Atlas need to pin their LangChain version to <=0.0.304
> 


## Related

- Vector store [conceptual guide](/docs/concepts/#vector-stores)
- Vector store [how-to guides](/docs/how_to/#vector-stores)
