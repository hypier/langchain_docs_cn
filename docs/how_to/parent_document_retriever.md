---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/parent_document_retriever.ipynb
---

# 如何使用父文档检索器

在进行文档拆分以便检索时，常常会有相互冲突的需求：

1. 你可能希望文档较小，以便其嵌入能够更准确地反映其含义。如果文档过长，则嵌入可能会失去意义。
2. 你希望文档足够长，以保留每个块的上下文。

`ParentDocumentRetriever`通过拆分和存储小块数据来实现这种平衡。在检索时，它首先获取小块，然后查找这些块的父ID并返回那些更大的文档。

请注意，“父文档”是指小块来源的文档。这可以是整个原始文档或更大的块。

```python
from langchain.retrievers import ParentDocumentRetriever
```

```python
from langchain.storage import InMemoryStore
from langchain_chroma import Chroma
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
```

```python
loaders = [
    TextLoader("paul_graham_essay.txt"),
    TextLoader("state_of_the_union.txt"),
]
docs = []
for loader in loaders:
    docs.extend(loader.load())
```

## 检索完整文档

在此模式下，我们希望检索完整文档。因此，我们只指定一个子分割器。


```python
# This text splitter is used to create the child documents
child_splitter = RecursiveCharacterTextSplitter(chunk_size=400)
# The vectorstore to use to index the child chunks
vectorstore = Chroma(
    collection_name="full_documents", embedding_function=OpenAIEmbeddings()
)
# The storage layer for the parent documents
store = InMemoryStore()
retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=store,
    child_splitter=child_splitter,
)
```


```python
retriever.add_documents(docs, ids=None)
```

这应该返回两个键，因为我们添加了两个文档。


```python
list(store.yield_keys())
```



```output
['9a63376c-58cc-42c9-b0f7-61f0e1a3a688',
 '40091598-e918-4a18-9be0-f46413a95ae4']
```


现在让我们调用向量存储搜索功能 - 我们应该看到它返回小块（因为我们存储的是小块）。


```python
sub_docs = vectorstore.similarity_search("justice breyer")
```


```python
print(sub_docs[0].page_content)
```
```output
Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court.
```
现在让我们从整体检索器中检索。这应该返回大文档 - 因为它返回小块所在的文档。


```python
retrieved_docs = retriever.invoke("justice breyer")
```


```python
len(retrieved_docs[0].page_content)
```



```output
38540
```

## 检索更大的块

有时候，完整的文档可能太大，不适合按原样检索。在这种情况下，我们真正想做的是先将原始文档拆分成更大的块，然后再拆分成更小的块。我们随后对更小的块进行索引，但在检索时我们检索更大的块（但仍然不是完整的文档）。

```python
# 这个文本分割器用于创建父文档
parent_splitter = RecursiveCharacterTextSplitter(chunk_size=2000)
# 这个文本分割器用于创建子文档
# 它应该创建比父文档更小的文档
child_splitter = RecursiveCharacterTextSplitter(chunk_size=400)
# 用于索引子块的向量存储
vectorstore = Chroma(
    collection_name="split_parents", embedding_function=OpenAIEmbeddings()
)
# 父文档的存储层
store = InMemoryStore()
```

```python
retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=store,
    child_splitter=child_splitter,
    parent_splitter=parent_splitter,
)
```

```python
retriever.add_documents(docs)
```

我们可以看到现在有比两个文档更多的文档——这些是更大的块。

```python
len(list(store.yield_keys()))
```

```output
66
```

让我们确保底层的向量存储仍然可以检索小块。

```python
sub_docs = vectorstore.similarity_search("justice breyer")
```

```python
print(sub_docs[0].page_content)
```
```output
今晚，我想表彰一位为这个国家奉献一生的人：史蒂芬·布雷耶法官——一位退伍军人、宪法学者，以及即将退休的美国最高法院法官。布雷耶法官，感谢您的服务。

总统最严肃的宪法责任之一是提名某人担任美国最高法院的法官。
```

```python
retrieved_docs = retriever.invoke("justice breyer")
```

```python
len(retrieved_docs[0].page_content)
```

```output
1849
```

```python
print(retrieved_docs[0].page_content)
```
```output
在一个又一个州，新法律相继通过，不仅压制投票，还颠覆整个选举。

我们不能让这种情况发生。

今晚，我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯投票权法案》。同时通过《披露法案》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：史蒂芬·布雷耶法官——一位退伍军人、宪法学者，以及即将退休的美国最高法院法官。布雷耶法官，感谢您的服务。

总统最严肃的宪法责任之一是提名某人担任美国最高法院的法官。

我在四天前做了这一点，当时我提名了上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律人才之一，将继续布雷耶法官卓越的遗产。

她曾是一名顶尖的私人执业律师。曾是一名联邦公设辩护人。并来自一家庭成员都是公立学校教育工作者和警察的家庭。她是一个共识的建立者。自从她被提名以来，她得到了广泛的支持——从兄弟警察组织到民主党和共和党任命的前法官。

如果我们要推进自由与正义，我们需要确保边界安全并修复移民系统。

我们可以同时做到这两点。在我们的边界，我们安装了新技术，比如尖端扫描仪，以更好地检测毒品走私。

我们与墨西哥和危地马拉建立了联合巡逻，以抓捕更多的人贩子。

我们正在安排专门的移民法官，以便逃离迫害和暴力的家庭能够更快地听取他们的案件。

我们正在确保承诺，并支持南美和中美洲的合作伙伴接纳更多难民并确保他们自己的边界安全。
```