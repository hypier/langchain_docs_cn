---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/astradb.ipynb
---
# Astra DB

This page provides a quickstart for using [Astra DB](https://docs.datastax.com/en/astra/home/astra.html) as a Vector Store.

> DataStax [Astra DB](https://docs.datastax.com/en/astra/home/astra.html) is a serverless vector-capable database built on Apache Cassandra® and made conveniently available through an easy-to-use JSON API.

_Note: in addition to access to the database, an OpenAI API Key is required to run the full example._

## Setup and general dependencies

Use of the integration requires the corresponding Python package:


```python
pip install -qU langchain-astradb
```

_Make sure you have installed the packages required to run all of this demo:_


```python
pip install -qU langchain langchain-community langchain-openai datasets pypdf
```

### Import dependencies


```python
import os
from getpass import getpass

from astrapy.info import CollectionVectorServiceOptions
from datasets import load_dataset
from langchain_community.document_loaders import PyPDFLoader
from langchain_core.documents import Document
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
```

## Import the Vector Store


```python
from langchain_astradb import AstraDBVectorStore
```

## DB Connection parameters

These are found on your Astra DB dashboard:

- the API Endpoint looks like `https://01234567-89ab-cdef-0123-456789abcdef-us-east1.apps.astra.datastax.com`
- the Token looks like `AstraCS:6gBhNmsk135....`
- you may optionally provide a _Namespace_ such as `my_namespace`


```python
ASTRA_DB_API_ENDPOINT = input("ASTRA_DB_API_ENDPOINT = ")
ASTRA_DB_APPLICATION_TOKEN = getpass("ASTRA_DB_APPLICATION_TOKEN = ")

desired_namespace = input("(optional) Namespace = ")
if desired_namespace:
    ASTRA_DB_KEYSPACE = desired_namespace
else:
    ASTRA_DB_KEYSPACE = None
```

## Create the vector store

There are two ways to create an Astra DB vector store, which differ in how the embeddings are computed.

*Explicit embeddings*. You can separately instantiate a `langchain_core.embeddings.Embeddings` class and pass it to the `AstraDBVectorStore` constructor, just like with most other LangChain vector stores.

*Integrated embedding computation*. Alternatively, you can use the [Vectorize](https://www.datastax.com/blog/simplifying-vector-embedding-generation-with-astra-vectorize) feature of Astra DB and simply specify the name of a supported embedding model when creating the store. The embedding computations are entirely handled within the database. (To proceed with this method, you must have enabled the desired embedding integration for your database, as described [in the docs](https://docs.datastax.com/en/astra-db-serverless/databases/embedding-generation.html).)

**Please choose one method and run the corresponding cells only.**

### Method 1: provide embeddings explicitly

This demo will use an OpenAI embedding model:


```python
os.environ["OPENAI_API_KEY"] = getpass("OPENAI_API_KEY = ")
```


```python
my_embeddings = OpenAIEmbeddings()
```

Now you can create the vector store:


```python
vstore = AstraDBVectorStore(
    embedding=my_embeddings,
    collection_name="astra_vector_demo",
    api_endpoint=ASTRA_DB_API_ENDPOINT,
    token=ASTRA_DB_APPLICATION_TOKEN,
    namespace=ASTRA_DB_KEYSPACE,
)
```

### Method 2: use Astra Vectorize (embeddings integrated in Astra DB)

Here it is assumed that you have

- enabled the OpenAI integration in your Astra DB organization,
-  added an API Key named `"MY_OPENAI_API_KEY"` to the integration, and
- scoped it to the database you are using.

For more details please consult the [documentation](https://docs.datastax.com/en/astra-db-serverless/integrations/embedding-providers/openai.html).


```python
openai_vectorize_options = CollectionVectorServiceOptions(
    provider="openai",
    model_name="text-embedding-3-small",
    authentication={
        "providerKey": "MY_OPENAI_API_KEY",
    },
)

vstore = AstraDBVectorStore(
    collection_name="astra_vectorize_demo",
    api_endpoint=ASTRA_DB_API_ENDPOINT,
    token=ASTRA_DB_APPLICATION_TOKEN,
    namespace=ASTRA_DB_KEYSPACE,
    collection_vector_service_options=openai_vectorize_options,
)
```

## Load a dataset

Convert each entry in the source dataset into a `Document`, then write them into the vector store:


```python
philo_dataset = load_dataset("datastax/philosopher-quotes")["train"]

docs = []
for entry in philo_dataset:
    metadata = {"author": entry["author"]}
    doc = Document(page_content=entry["quote"], metadata=metadata)
    docs.append(doc)

inserted_ids = vstore.add_documents(docs)
print(f"\nInserted {len(inserted_ids)} documents.")
```

In the above, `metadata` dictionaries are created from the source data and are part of the `Document`.

_Note: check the [Astra DB API Docs](https://docs.datastax.com/en/astra-serverless/docs/develop/dev-with-json.html#_json_api_limits) for the valid metadata field names: some characters are reserved and cannot be used._

Add some more entries, this time with `add_texts`:


```python
texts = ["I think, therefore I am.", "To the things themselves!"]
metadatas = [{"author": "descartes"}, {"author": "husserl"}]
ids = ["desc_01", "huss_xy"]

inserted_ids_2 = vstore.add_texts(texts=texts, metadatas=metadatas, ids=ids)
print(f"\nInserted {len(inserted_ids_2)} documents.")
```

_Note: you may want to speed up the execution of `add_texts` and `add_documents` by increasing the concurrency level for_
_these bulk operations - check out the `*_concurrency` parameters in the class constructor and the `add_texts` docstrings_
_for more details. Depending on the network and the client machine specifications, your best-performing choice of parameters may vary._

## Run searches

This section demonstrates metadata filtering and getting the similarity scores back:


```python
results = vstore.similarity_search("Our life is what we make of it", k=3)
for res in results:
    print(f"* {res.page_content} [{res.metadata}]")
```


```python
results_filtered = vstore.similarity_search(
    "Our life is what we make of it",
    k=3,
    filter={"author": "plato"},
)
for res in results_filtered:
    print(f"* {res.page_content} [{res.metadata}]")
```


```python
results = vstore.similarity_search_with_score("Our life is what we make of it", k=3)
for res, score in results:
    print(f"* [SIM={score:3f}] {res.page_content} [{res.metadata}]")
```

### MMR (Maximal-marginal-relevance) search

_Note: the MMR search method is not (yet) supported for vector stores built with Astra Vectorize._


```python
results = vstore.max_marginal_relevance_search(
    "Our life is what we make of it",
    k=3,
    filter={"author": "aristotle"},
)
for res in results:
    print(f"* {res.page_content} [{res.metadata}]")
```

### Async

Note that the Astra DB vector store supports all fully async methods (`asimilarity_search`, `afrom_texts`, `adelete` and so on) natively, i.e. without thread wrapping involved.

## Deleting stored documents


```python
delete_1 = vstore.delete(inserted_ids[:3])
print(f"all_succeed={delete_1}")  # True, all documents deleted
```


```python
delete_2 = vstore.delete(inserted_ids[2:5])
print(f"some_succeeds={delete_2}")  # True, though some IDs were gone already
```

## A minimal RAG chain

The next cells will implement a simple RAG pipeline:
- download a sample PDF file and load it onto the store;
- create a RAG chain with LCEL (LangChain Expression Language), with the vector store at its heart;
- run the question-answering chain.


```python
!curl -L \
    "https://github.com/awesome-astra/datasets/blob/main/demo-resources/what-is-philosophy/what-is-philosophy.pdf?raw=true" \
    -o "what-is-philosophy.pdf"
```


```python
pdf_loader = PyPDFLoader("what-is-philosophy.pdf")
splitter = RecursiveCharacterTextSplitter(chunk_size=512, chunk_overlap=64)
docs_from_pdf = pdf_loader.load_and_split(text_splitter=splitter)

print(f"Documents from PDF: {len(docs_from_pdf)}.")
inserted_ids_from_pdf = vstore.add_documents(docs_from_pdf)
print(f"Inserted {len(inserted_ids_from_pdf)} documents.")
```


```python
retriever = vstore.as_retriever(search_kwargs={"k": 3})

philo_template = """
You are a philosopher that draws inspiration from great thinkers of the past
to craft well-thought answers to user questions. Use the provided context as the basis
for your answers and do not make up new reasoning paths - just mix-and-match what you are given.
Your answers must be concise and to the point, and refrain from answering about other topics than philosophy.

CONTEXT:
{context}

QUESTION: {question}

YOUR ANSWER:"""

philo_prompt = ChatPromptTemplate.from_template(philo_template)

llm = ChatOpenAI()

chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | philo_prompt
    | llm
    | StrOutputParser()
)
```


```python
chain.invoke("How does Russel elaborate on Peirce's idea of the security blanket?")
```

For more, check out a complete RAG template using Astra DB [here](https://github.com/langchain-ai/langchain/tree/master/templates/rag-astradb).

## Cleanup

If you want to completely delete the collection from your Astra DB instance, run this.

_(You will lose the data you stored in it.)_


```python
vstore.delete_collection()
```


## Related

- Vector store [conceptual guide](/docs/concepts/#vector-stores)
- Vector store [how-to guides](/docs/how_to/#vector-stores)
