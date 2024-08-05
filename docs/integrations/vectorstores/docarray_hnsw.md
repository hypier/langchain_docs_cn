---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/docarray_hnsw.ipynb
---

# DocArray HnswSearch

>[DocArrayHnswSearch](https://docs.docarray.org/user_guide/storing/index_hnswlib/) 是一个由 [Docarray](https://github.com/docarray/docarray) 提供的轻量级文档索引实现，完全在本地运行，最适合小型到中型数据集。它在 [hnswlib](https://github.com/nmslib/hnswlib) 中将向量存储在磁盘上，并在 [SQLite](https://www.sqlite.org/index.html) 中存储所有其他数据。

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 才能使用此集成。

本笔记本展示了如何使用与 `DocArrayHnswSearch` 相关的功能。

## 设置

取消注释以下单元以安装 docarray，并在尚未完成的情况下获取/设置您的 OpenAI api 密钥。


```python
%pip install --upgrade --quiet  "docarray[hnswlib]"
```


```python
# 获取 OpenAI 令牌: https://platform.openai.com/account/api-keys

# import os
# from getpass import getpass

# OPENAI_API_KEY = getpass()

# os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

## 使用 DocArrayHnswSearch


```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import DocArrayHnswSearch
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```


```python
documents = TextLoader("../../how_to/state_of_the_union.txt").load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()

db = DocArrayHnswSearch.from_documents(
    docs, embeddings, work_dir="hnswlib_store/", n_dim=1536
)
```

### 相似性搜索


```python
query = "总统关于凯坦吉·布朗·杰克逊说了什么"
docs = db.similarity_search(query)
```


```python
print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《投票自由法》。通过《约翰·刘易斯投票权法》。同时，通过《披露法》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：史蒂芬·布雷耶大法官——一位退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最严肃的宪法责任之一就是提名某人担任美国最高法院大法官。

而我在4天前做到了这一点，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官卓越的遗产。
```

### 相似性搜索与评分

返回的距离分数是余弦距离。因此，分数越低越好。


```python
docs = db.similarity_search_with_score(query)
```


```python
docs[0]
```



```output
(Document(page_content='今晚。我呼吁参议院：通过投票自由法案。通过约翰·刘易斯投票权法案。并且在此期间，通过披露法案，让美国人知道谁在资助我们的选举。\n\n今晚，我想向一位为这个国家奉献一生的人致敬：大法官斯蒂芬·布雷耶——一位陆军退伍军人，宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。\n\n总统最重要的宪法责任之一是提名某人担任美国最高法院法官。\n\n我在4天前做到了这一点，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律思想家之一，将继续布雷耶大法官卓越的遗产。', metadata={}),
 0.36962226)
```



```python
import shutil

# delete the dir
shutil.rmtree("hnswlib_store")
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)