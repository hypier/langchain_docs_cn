---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/duckdb.ipynb
---
# DuckDB
This notebook shows how to use `DuckDB` as a vector store.


```python
! pip install duckdb langchain langchain-community langchain-openai
```

We want to use OpenAIEmbeddings so we have to get the OpenAI API Key. 


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```


```python
from langchain_community.vectorstores import DuckDB
from langchain_openai import OpenAIEmbeddings
```


```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()

documents = CharacterTextSplitter().split_documents(documents)
embeddings = OpenAIEmbeddings()
```


```python
docsearch = DuckDB.from_documents(documents, embeddings)

query = "What did the president say about Ketanji Brown Jackson"
docs = docsearch.similarity_search(query)
```


```python
print(docs[0].page_content)
```


## Related

- Vector store [conceptual guide](/docs/concepts/#vector-stores)
- Vector store [how-to guides](/docs/how_to/#vector-stores)
