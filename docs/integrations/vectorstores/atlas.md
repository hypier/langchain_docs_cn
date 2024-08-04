---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/atlas.ipynb
---
# Atlas


>[Atlas](https://docs.nomic.ai/index.html) is a platform by Nomic made for interacting with both small and internet scale unstructured datasets. It enables anyone to visualize, search, and share massive datasets in their browser.

You'll need to install `langchain-community` with `pip install -qU langchain-community` to use this integration

This notebook shows you how to use functionality related to the `AtlasDB` vectorstore.


```python
%pip install --upgrade --quiet  spacy
```


```python
!python3 -m spacy download en_core_web_sm
```


```python
%pip install --upgrade --quiet  nomic
```

### Load Packages


```python
import time

from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import AtlasDB
from langchain_text_splitters import SpacyTextSplitter
```


```python
ATLAS_TEST_API_KEY = "7xDPkYXSYDc1_ErdTPIcoAR9RNd8YDlkS3nVNXcVoIMZ6"
```

### Prepare the Data


```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = SpacyTextSplitter(separator="|")
texts = []
for doc in text_splitter.split_documents(documents):
    texts.extend(doc.page_content.split("|"))

texts = [e.strip() for e in texts]
```

### Map the Data using Nomic's Atlas


```python
db = AtlasDB.from_texts(
    texts=texts,
    name="test_index_" + str(time.time()),  # unique name for your vector store
    description="test_index",  # a description for your vector store
    api_key=ATLAS_TEST_API_KEY,
    index_kwargs={"build_topic_model": True},
)
```


```python
db.project.wait_for_project_lock()
```


```python
db.project
```

Here is a map with the result of this code. This map displays the texts of the State of the Union.
https://atlas.nomic.ai/map/3e4de075-89ff-486a-845c-36c23f30bb67/d8ce2284-8edb-4050-8b9b-9bb543d7f647


## Related

- Vector store [conceptual guide](/docs/concepts/#vector-stores)
- Vector store [how-to guides](/docs/how_to/#vector-stores)