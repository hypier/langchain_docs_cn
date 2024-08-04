---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/faiss.ipynb
---
# Faiss

>[Facebook AI Similarity Search (Faiss)](https://engineering.fb.com/2017/03/29/data-infrastructure/faiss-a-library-for-efficient-similarity-search/) is a library for efficient similarity search and clustering of dense vectors. It contains algorithms that search in sets of vectors of any size, up to ones that possibly do not fit in RAM. It also contains supporting code for evaluation and parameter tuning.

[Faiss documentation](https://faiss.ai/).

You'll need to install `langchain-community` with `pip install -qU langchain-community` to use this integration

This notebook shows how to use functionality related to the `FAISS` vector database. It will show functionality specific to this integration. After going through, it may be useful to explore [relevant use-case pages](/docs/how_to#qa-with-rag) to learn how to use this vectorstore as part of a larger chain.

## Setup

The integration lives in the `langchain-community` package. We also need to install the `faiss` package itself. We will also be using OpenAI for embeddings, so we need to install those requirements. We can install these with:

```bash
pip install -U langchain-community faiss-cpu langchain-openai tiktoken
```

Note that you can also install `faiss-gpu` if you want to use the GPU enabled version

Since we are using OpenAI, you will need an OpenAI API Key.


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()
```

It's also helpful (but not needed) to set up [LangSmith](https://smith.langchain.com/) for best-in-class observability


```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## Ingestion

Here, we ingest documents into the vectorstore


```python
# Uncomment the following line if you need to initialize FAISS with no AVX2 optimization
# os.environ['FAISS_NO_AVX2'] = '1'

from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
embeddings = OpenAIEmbeddings()
db = FAISS.from_documents(docs, embeddings)
print(db.index.ntotal)
```



```output
42
```


## Querying

Now, we can query the vectorstore. There a few methods to do this. The most standard is to use `similarity_search`.


```python
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
```


```python
print(docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```
## As a Retriever

We can also convert the vectorstore into a [Retriever](/docs/how_to#retrievers) class. This allows us to easily use it in other LangChain methods, which largely work with retrievers


```python
retriever = db.as_retriever()
docs = retriever.invoke(query)
```


```python
print(docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```
## Similarity Search with score
There are some FAISS specific methods. One of them is `similarity_search_with_score`, which allows you to return not only the documents but also the distance score of the query to them. The returned distance score is L2 distance. Therefore, a lower score is better.


```python
docs_and_scores = db.similarity_search_with_score(query)
```


```python
docs_and_scores[0]
```



```output
(Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../how_to/state_of_the_union.txt'}),
 0.36913747)
```


It is also possible to do a search for documents similar to a given embedding vector using `similarity_search_by_vector` which accepts an embedding vector as a parameter instead of a string.


```python
embedding_vector = embeddings.embed_query(query)
docs_and_scores = db.similarity_search_by_vector(embedding_vector)
```

## Saving and loading
You can also save and load a FAISS index. This is useful so you don't have to recreate it everytime you use it.


```python
db.save_local("faiss_index")

new_db = FAISS.load_local("faiss_index", embeddings)

docs = new_db.similarity_search(query)
```


```python
docs[0]
```



```output
Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../../state_of_the_union.txt'})
```


# Serializing and De-Serializing to bytes

you can pickle the FAISS Index by these functions. If you use embeddings model which is of 90 mb (sentence-transformers/all-MiniLM-L6-v2 or any other model), the resultant pickle size would be more than 90 mb. the size of the model is also included in the overall size. To overcome this, use the below functions. These functions only serializes FAISS index and size would be much lesser. this can be helpful if you wish to store the index in database like sql.


```python
from langchain_huggingface import HuggingFaceEmbeddings

pkl = db.serialize_to_bytes()  # serializes the faiss
embeddings = HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")

db = FAISS.deserialize_from_bytes(
    embeddings=embeddings, serialized=pkl
)  # Load the index
```

## Merging
You can also merge two FAISS vectorstores


```python
db1 = FAISS.from_texts(["foo"], embeddings)
db2 = FAISS.from_texts(["bar"], embeddings)

db1.docstore._dict
```


```python
db2.docstore._dict
```



```output
{'807e0c63-13f6-4070-9774-5c6f0fbb9866': Document(page_content='bar', metadata={})}
```



```python
db1.merge_from(db2)
```


```python
db1.docstore._dict
```



```output
{'068c473b-d420-487a-806b-fb0ccea7f711': Document(page_content='foo', metadata={}),
 '807e0c63-13f6-4070-9774-5c6f0fbb9866': Document(page_content='bar', metadata={})}
```


## Similarity Search with filtering
FAISS vectorstore can also support filtering, since the FAISS does not natively support filtering we have to do it manually. This is done by first fetching more results than `k` and then filtering them. This filter is either a callble that takes as input a metadata dict and returns a bool, or a metadata dict where each missing key is ignored and each present k must be in a list of values. You can also set the `fetch_k` parameter when calling any search method to set how many documents you want to fetch before filtering. Here is a small example:


```python
from langchain_core.documents import Document

list_of_documents = [
    Document(page_content="foo", metadata=dict(page=1)),
    Document(page_content="bar", metadata=dict(page=1)),
    Document(page_content="foo", metadata=dict(page=2)),
    Document(page_content="barbar", metadata=dict(page=2)),
    Document(page_content="foo", metadata=dict(page=3)),
    Document(page_content="bar burr", metadata=dict(page=3)),
    Document(page_content="foo", metadata=dict(page=4)),
    Document(page_content="bar bruh", metadata=dict(page=4)),
]
db = FAISS.from_documents(list_of_documents, embeddings)
results_with_scores = db.similarity_search_with_score("foo")
for doc, score in results_with_scores:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}, Score: {score}")
```
```output
Content: foo, Metadata: {'page': 1}, Score: 5.159960813797904e-15
Content: foo, Metadata: {'page': 2}, Score: 5.159960813797904e-15
Content: foo, Metadata: {'page': 3}, Score: 5.159960813797904e-15
Content: foo, Metadata: {'page': 4}, Score: 5.159960813797904e-15
```
Now we make the same query call but we filter for only `page = 1` 


```python
results_with_scores = db.similarity_search_with_score("foo", filter=dict(page=1))
# Or with a callable:
# results_with_scores = db.similarity_search_with_score("foo", filter=lambda d: d["page"] == 1)
for doc, score in results_with_scores:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}, Score: {score}")
```
```output
Content: foo, Metadata: {'page': 1}, Score: 5.159960813797904e-15
Content: bar, Metadata: {'page': 1}, Score: 0.3131446838378906
```
Same thing can be done with the `max_marginal_relevance_search` as well.


```python
results = db.max_marginal_relevance_search("foo", filter=dict(page=1))
for doc in results:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}")
```
```output
Content: foo, Metadata: {'page': 1}
Content: bar, Metadata: {'page': 1}
```
Here is an example of how to set `fetch_k` parameter when calling `similarity_search`. Usually you would want the `fetch_k` parameter >> `k` parameter. This is because the `fetch_k` parameter is the number of documents that will be fetched before filtering. If you set `fetch_k` to a low number, you might not get enough documents to filter from.


```python
results = db.similarity_search("foo", filter=dict(page=1), k=1, fetch_k=4)
for doc in results:
    print(f"Content: {doc.page_content}, Metadata: {doc.metadata}")
```
```output
Content: foo, Metadata: {'page': 1}
```
## Delete

You can also delete records from vectorstore. In the example below `db.index_to_docstore_id` represents a dictionary with elements of the FAISS index.


```python
print("count before:", db.index.ntotal)
db.delete([db.index_to_docstore_id[0]])
print("count after:", db.index.ntotal)
```



```output
count before: 8
count after: 7
```



## Related

- Vector store [conceptual guide](/docs/concepts/#vector-stores)
- Vector store [how-to guides](/docs/how_to/#vector-stores)
