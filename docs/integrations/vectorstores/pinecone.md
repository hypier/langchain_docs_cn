---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/pinecone.ipynb
---
# Pinecone

>[Pinecone](https://docs.pinecone.io/docs/overview) is a vector database with broad functionality.

This notebook shows how to use functionality related to the `Pinecone` vector database.

Set the following environment variables to follow along in this doc:
- `OPENAI_API_KEY`: Your OpenAI API key, for using `OpenAIEmbeddings`


```python
%pip install --upgrade --quiet  \
    langchain-pinecone \
    langchain-openai \
    langchain \
    langchain-community \
    pinecone-notebooks
```

Migration note: if you are migrating from the `langchain_community.vectorstores` implementation of Pinecone, you may need to remove your `pinecone-client` v2 dependency before installing `langchain-pinecone`, which relies on `pinecone-client` v3.

First, let's split our state of the union document into chunked `docs`.


```python
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```

Now let's create a new Pinecone account, or sign into your existing one, and create an API key to use in this notebook.


```python
from pinecone_notebooks.colab import Authenticate

Authenticate()
```

The newly created API key has been stored in the `PINECONE_API_KEY` environment variable. We will use it to setup the Pinecone client.


```python
import os

pinecone_api_key = os.environ.get("PINECONE_API_KEY")
pinecone_api_key

import time

from pinecone import Pinecone, ServerlessSpec

pc = Pinecone(api_key=pinecone_api_key)
```

Next, let's connect to your Pinecone index. If one named `index_name` doesn't exist, it will be created.


```python
import time

index_name = "langchain-index"  # change if desired

existing_indexes = [index_info["name"] for index_info in pc.list_indexes()]

if index_name not in existing_indexes:
    pc.create_index(
        name=index_name,
        dimension=1536,
        metric="cosine",
        spec=ServerlessSpec(cloud="aws", region="us-east-1"),
    )
    while not pc.describe_index(index_name).status["ready"]:
        time.sleep(1)

index = pc.Index(index_name)
```

Now that our Pinecone index is setup, we can upsert those chunked docs as contents with `PineconeVectorStore.from_documents`.


```python
from langchain_pinecone import PineconeVectorStore

docsearch = PineconeVectorStore.from_documents(docs, embeddings, index_name=index_name)
```


```python
query = "What did the president say about Ketanji Brown Jackson"
docs = docsearch.similarity_search(query)
print(docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```
### Adding More Text to an Existing Index

More text can embedded and upserted to an existing Pinecone index using the `add_texts` function



```python
vectorstore = PineconeVectorStore(index_name=index_name, embedding=embeddings)

vectorstore.add_texts(["More text!"])
```



```output
['24631802-4bad-44a7-a4ba-fd71f00cc160']
```


### Maximal Marginal Relevance Searches

In addition to using similarity search in the retriever object, you can also use `mmr` as retriever.



```python
retriever = docsearch.as_retriever(search_type="mmr")
matched_docs = retriever.invoke(query)
for i, d in enumerate(matched_docs):
    print(f"\n## Document {i}\n")
    print(d.page_content)
```
```output

## Document 0

Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.

## Document 1

And I’m taking robust action to make sure the pain of our sanctions  is targeted at Russia’s economy. And I will use every tool at our disposal to protect American businesses and consumers. 

Tonight, I can announce that the United States has worked with 30 other countries to release 60 Million barrels of oil from reserves around the world.  

America will lead that effort, releasing 30 Million barrels from our own Strategic Petroleum Reserve. And we stand ready to do more if necessary, unified with our allies.  

These steps will help blunt gas prices here at home. And I know the news about what’s happening can seem alarming. 

But I want you to know that we are going to be okay. 

When the history of this era is written Putin’s war on Ukraine will have left Russia weaker and the rest of the world stronger. 

While it shouldn’t have taken something so terrible for people around the world to see what’s at stake now everyone sees it clearly.

## Document 2

We can’t change how divided we’ve been. But we can change how we move forward—on COVID-19 and other issues we must face together. 

I recently visited the New York City Police Department days after the funerals of Officer Wilbert Mora and his partner, Officer Jason Rivera. 

They were responding to a 9-1-1 call when a man shot and killed them with a stolen gun. 

Officer Mora was 27 years old. 

Officer Rivera was 22. 

Both Dominican Americans who’d grown up on the same streets they later chose to patrol as police officers. 

I spoke with their families and told them that we are forever in debt for their sacrifice, and we will carry on their mission to restore the trust and safety every community deserves. 

I’ve worked on these issues a long time. 

I know what works: Investing in crime prevention and community police officers who’ll walk the beat, who’ll know the neighborhood, and who can restore trust and safety.

## Document 3

One was stationed at bases and breathing in toxic smoke from “burn pits” that incinerated wastes of war—medical and hazard material, jet fuel, and more. 

When they came home, many of the world’s fittest and best trained warriors were never the same. 

Headaches. Numbness. Dizziness. 

A cancer that would put them in a flag-draped coffin. 

I know. 

One of those soldiers was my son Major Beau Biden. 

We don’t know for sure if a burn pit was the cause of his brain cancer, or the diseases of so many of our troops. 

But I’m committed to finding out everything we can. 

Committed to military families like Danielle Robinson from Ohio. 

The widow of Sergeant First Class Heath Robinson.  

He was born a soldier. Army National Guard. Combat medic in Kosovo and Iraq. 

Stationed near Baghdad, just yards from burn pits the size of football fields. 

Heath’s widow Danielle is here with us tonight. They loved going to Ohio State football games. He loved building Legos with their daughter.
```
Or use `max_marginal_relevance_search` directly:


```python
found_docs = docsearch.max_marginal_relevance_search(query, k=2, fetch_k=10)
for i, doc in enumerate(found_docs):
    print(f"{i + 1}.", doc.page_content, "\n")
```
```output
1. Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence. 

2. We can’t change how divided we’ve been. But we can change how we move forward—on COVID-19 and other issues we must face together. 

I recently visited the New York City Police Department days after the funerals of Officer Wilbert Mora and his partner, Officer Jason Rivera. 

They were responding to a 9-1-1 call when a man shot and killed them with a stolen gun. 

Officer Mora was 27 years old. 

Officer Rivera was 22. 

Both Dominican Americans who’d grown up on the same streets they later chose to patrol as police officers. 

I spoke with their families and told them that we are forever in debt for their sacrifice, and we will carry on their mission to restore the trust and safety every community deserves. 

I’ve worked on these issues a long time. 

I know what works: Investing in crime prevention and community police officers who’ll walk the beat, who’ll know the neighborhood, and who can restore trust and safety.
```

## Related

- Vector store [conceptual guide](/docs/concepts/#vector-stores)
- Vector store [how-to guides](/docs/how_to/#vector-stores)
