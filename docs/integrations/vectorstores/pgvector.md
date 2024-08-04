---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/pgvector.ipynb
---
# PGVector

> An implementation of LangChain vectorstore abstraction using `postgres` as the backend and utilizing the `pgvector` extension.

The code lives in an integration package called: [langchain_postgres](https://github.com/langchain-ai/langchain-postgres/).

You can run the following command to spin up a a postgres container with the `pgvector` extension:

```shell
docker run --name pgvector-container -e POSTGRES_USER=langchain -e POSTGRES_PASSWORD=langchain -e POSTGRES_DB=langchain -p 6024:5432 -d pgvector/pgvector:pg16
```

## Status

This code has been ported over from `langchain_community` into a dedicated package called `langchain-postgres`. The following changes have been made:

* langchain_postgres works only with psycopg3. Please update your connnecion strings from `postgresql+psycopg2://...` to `postgresql+psycopg://langchain:langchain@...` (yes, it's the driver name is `psycopg` not `psycopg3`, but it'll use `psycopg3`.
* The schema of the embedding store and collection have been changed to make add_documents work correctly with user specified ids.
* One has to pass an explicit connection object now.


Currently, there is **no mechanism** that supports easy data migration on schema changes. So any schema changes in the vectorstore will require the user to recreate the tables and re-add the documents.
If this is a concern, please use a different vectorstore. If not, this implementation should be fine for your use case.

## Install dependencies

Here, we're using `langchain_cohere` for embeddings, but you can use other embeddings providers.


```python
!pip install --quiet -U langchain_cohere
!pip install --quiet -U langchain_postgres
```

## Initialize the vectorstore


```python
from langchain_cohere import CohereEmbeddings
from langchain_core.documents import Document
from langchain_postgres import PGVector
from langchain_postgres.vectorstores import PGVector

# See docker command above to launch a postgres instance with pgvector enabled.
connection = "postgresql+psycopg://langchain:langchain@localhost:6024/langchain"  # Uses psycopg3!
collection_name = "my_docs"
embeddings = CohereEmbeddings(model="embed-english-v3.0")

vectorstore = PGVector(
    embeddings=embeddings,
    collection_name=collection_name,
    connection=connection,
    use_jsonb=True,
)
```

## Drop tables

If you need to drop tables (e.g., updating the embedding to a different dimension or just updating the embedding provider): 

```python
vectorstore.drop_tables()
````

## Add documents

Add documents to the vectorstore


```python
docs = [
    Document(
        page_content="there are cats in the pond",
        metadata={"id": 1, "location": "pond", "topic": "animals"},
    ),
    Document(
        page_content="ducks are also found in the pond",
        metadata={"id": 2, "location": "pond", "topic": "animals"},
    ),
    Document(
        page_content="fresh apples are available at the market",
        metadata={"id": 3, "location": "market", "topic": "food"},
    ),
    Document(
        page_content="the market also sells fresh oranges",
        metadata={"id": 4, "location": "market", "topic": "food"},
    ),
    Document(
        page_content="the new art exhibit is fascinating",
        metadata={"id": 5, "location": "museum", "topic": "art"},
    ),
    Document(
        page_content="a sculpture exhibit is also at the museum",
        metadata={"id": 6, "location": "museum", "topic": "art"},
    ),
    Document(
        page_content="a new coffee shop opened on Main Street",
        metadata={"id": 7, "location": "Main Street", "topic": "food"},
    ),
    Document(
        page_content="the book club meets at the library",
        metadata={"id": 8, "location": "library", "topic": "reading"},
    ),
    Document(
        page_content="the library hosts a weekly story time for kids",
        metadata={"id": 9, "location": "library", "topic": "reading"},
    ),
    Document(
        page_content="a cooking class for beginners is offered at the community center",
        metadata={"id": 10, "location": "community center", "topic": "classes"},
    ),
]
```


```python
vectorstore.add_documents(docs, ids=[doc.metadata["id"] for doc in docs])
```



```output
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```



```python
vectorstore.similarity_search("kitty", k=10)
```



```output
[Document(page_content='there are cats in the pond', metadata={'id': 1, 'topic': 'animals', 'location': 'pond'}),
 Document(page_content='the book club meets at the library', metadata={'id': 8, 'topic': 'reading', 'location': 'library'}),
 Document(page_content='the library hosts a weekly story time for kids', metadata={'id': 9, 'topic': 'reading', 'location': 'library'}),
 Document(page_content='the new art exhibit is fascinating', metadata={'id': 5, 'topic': 'art', 'location': 'museum'}),
 Document(page_content='ducks are also found in the pond', metadata={'id': 2, 'topic': 'animals', 'location': 'pond'}),
 Document(page_content='the market also sells fresh oranges', metadata={'id': 4, 'topic': 'food', 'location': 'market'}),
 Document(page_content='a cooking class for beginners is offered at the community center', metadata={'id': 10, 'topic': 'classes', 'location': 'community center'}),
 Document(page_content='fresh apples are available at the market', metadata={'id': 3, 'topic': 'food', 'location': 'market'}),
 Document(page_content='a sculpture exhibit is also at the museum', metadata={'id': 6, 'topic': 'art', 'location': 'museum'}),
 Document(page_content='a new coffee shop opened on Main Street', metadata={'id': 7, 'topic': 'food', 'location': 'Main Street'})]
```


Adding documents by ID will over-write any existing documents that match that ID.


```python
docs = [
    Document(
        page_content="there are cats in the pond",
        metadata={"id": 1, "location": "pond", "topic": "animals"},
    ),
    Document(
        page_content="ducks are also found in the pond",
        metadata={"id": 2, "location": "pond", "topic": "animals"},
    ),
    Document(
        page_content="fresh apples are available at the market",
        metadata={"id": 3, "location": "market", "topic": "food"},
    ),
    Document(
        page_content="the market also sells fresh oranges",
        metadata={"id": 4, "location": "market", "topic": "food"},
    ),
    Document(
        page_content="the new art exhibit is fascinating",
        metadata={"id": 5, "location": "museum", "topic": "art"},
    ),
    Document(
        page_content="a sculpture exhibit is also at the museum",
        metadata={"id": 6, "location": "museum", "topic": "art"},
    ),
    Document(
        page_content="a new coffee shop opened on Main Street",
        metadata={"id": 7, "location": "Main Street", "topic": "food"},
    ),
    Document(
        page_content="the book club meets at the library",
        metadata={"id": 8, "location": "library", "topic": "reading"},
    ),
    Document(
        page_content="the library hosts a weekly story time for kids",
        metadata={"id": 9, "location": "library", "topic": "reading"},
    ),
    Document(
        page_content="a cooking class for beginners is offered at the community center",
        metadata={"id": 10, "location": "community center", "topic": "classes"},
    ),
]
```

## Filtering Support

The vectorstore supports a set of filters that can be applied against the metadata fields of the documents.

| Operator | Meaning/Category        |
|----------|-------------------------|
| \$eq      | Equality (==)           |
| \$ne      | Inequality (!=)         |
| \$lt      | Less than (<)           |
| \$lte     | Less than or equal (<=) |
| \$gt      | Greater than (>)        |
| \$gte     | Greater than or equal (>=) |
| \$in      | Special Cased (in)      |
| \$nin     | Special Cased (not in)  |
| \$between | Special Cased (between) |
| \$like    | Text (like)             |
| \$ilike   | Text (case-insensitive like) |
| \$and     | Logical (and)           |
| \$or      | Logical (or)            |


```python
vectorstore.similarity_search("kitty", k=10, filter={"id": {"$in": [1, 5, 2, 9]}})
```



```output
[Document(page_content='there are cats in the pond', metadata={'id': 1, 'topic': 'animals', 'location': 'pond'}),
 Document(page_content='the library hosts a weekly story time for kids', metadata={'id': 9, 'topic': 'reading', 'location': 'library'}),
 Document(page_content='the new art exhibit is fascinating', metadata={'id': 5, 'topic': 'art', 'location': 'museum'}),
 Document(page_content='ducks are also found in the pond', metadata={'id': 2, 'topic': 'animals', 'location': 'pond'})]
```


If you provide a dict with multiple fields, but no operators, the top level will be interpreted as a logical **AND** filter


```python
vectorstore.similarity_search(
    "ducks",
    k=10,
    filter={"id": {"$in": [1, 5, 2, 9]}, "location": {"$in": ["pond", "market"]}},
)
```



```output
[Document(page_content='ducks are also found in the pond', metadata={'id': 2, 'topic': 'animals', 'location': 'pond'}),
 Document(page_content='there are cats in the pond', metadata={'id': 1, 'topic': 'animals', 'location': 'pond'})]
```



```python
vectorstore.similarity_search(
    "ducks",
    k=10,
    filter={
        "$and": [
            {"id": {"$in": [1, 5, 2, 9]}},
            {"location": {"$in": ["pond", "market"]}},
        ]
    },
)
```



```output
[Document(page_content='ducks are also found in the pond', metadata={'id': 2, 'topic': 'animals', 'location': 'pond'}),
 Document(page_content='there are cats in the pond', metadata={'id': 1, 'topic': 'animals', 'location': 'pond'})]
```



```python
vectorstore.similarity_search("bird", k=10, filter={"location": {"$ne": "pond"}})
```



```output
[Document(page_content='the book club meets at the library', metadata={'id': 8, 'topic': 'reading', 'location': 'library'}),
 Document(page_content='the new art exhibit is fascinating', metadata={'id': 5, 'topic': 'art', 'location': 'museum'}),
 Document(page_content='the library hosts a weekly story time for kids', metadata={'id': 9, 'topic': 'reading', 'location': 'library'}),
 Document(page_content='a sculpture exhibit is also at the museum', metadata={'id': 6, 'topic': 'art', 'location': 'museum'}),
 Document(page_content='the market also sells fresh oranges', metadata={'id': 4, 'topic': 'food', 'location': 'market'}),
 Document(page_content='a cooking class for beginners is offered at the community center', metadata={'id': 10, 'topic': 'classes', 'location': 'community center'}),
 Document(page_content='a new coffee shop opened on Main Street', metadata={'id': 7, 'topic': 'food', 'location': 'Main Street'}),
 Document(page_content='fresh apples are available at the market', metadata={'id': 3, 'topic': 'food', 'location': 'market'})]
```



## Related

- Vector store [conceptual guide](/docs/concepts/#vector-stores)
- Vector store [how-to guides](/docs/how_to/#vector-stores)
