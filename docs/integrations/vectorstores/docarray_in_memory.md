---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/docarray_in_memory.ipynb
---

# DocArray InMemorySearch

>[DocArrayInMemorySearch](https://docs.docarray.org/user_guide/storing/index_in_memory/) 是一个由 [Docarray](https://github.com/docarray/docarray) 提供的文档索引，它将文档存储在内存中。对于小型数据集，这是一个很好的起点，因为您可能不想启动一个数据库服务器。

本笔记本展示了如何使用与 `DocArrayInMemorySearch` 相关的功能。

## 设置

如果您尚未这样做，请取消注释以下单元格以安装 docarray 并获取/设置您的 OpenAI api 密钥。


```python
%pip install --upgrade --quiet  langchain-community "docarray"
```


```python
# 获取 OpenAI 令牌: https://platform.openai.com/account/api-keys

# import os
# from getpass import getpass

# OPENAI_API_KEY = getpass()

# os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

## 使用 DocArrayInMemorySearch


```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import DocArrayInMemorySearch
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```


```python
documents = TextLoader("../../how_to/state_of_the_union.txt").load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()

db = DocArrayInMemorySearch.from_documents(docs, embeddings)
```

### 相似性搜索


```python
query = "总统对凯坦吉·布朗·杰克逊说了什么"
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

### 带分数的相似性搜索

返回的距离分数是余弦距离。因此，分数越低越好。


```python
docs = db.similarity_search_with_score(query)
```


```python
docs[0]
```



```output
(Document(page_content='今晚。我呼吁参议院：通过投票自由法案。通过约翰·刘易斯投票权法案。而且在此期间，通过披露法案，让美国人知道是谁在资助我们的选举。\n\n今晚，我想表彰一位为这个国家奉献一生的人：斯蒂芬·布雷耶大法官——一位陆军退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。\n\n总统最严肃的宪法责任之一是提名某人担任美国最高法院大法官。\n\n而我在4天前做到了这一点，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官卓越的遗产。', metadata={}),
 0.8154190158347903)
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)