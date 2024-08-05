---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/bagel.ipynb
---

# 贝果

> [贝果](https://www.bagel.net/) (`Open Inference platform for AI`)，类似于 AI 数据的 GitHub。  
这是一个协作平台，用户可以创建、共享和管理推理数据集。它可以支持独立开发者的私有项目、企业内部合作以及数据 DAO 的公共贡献。

### 安装与设置

```bash
pip install bagelML langchain-community
```

## 从文本创建 VectorStore


```python
from langchain_community.vectorstores import Bagel

texts = ["hello bagel", "hello langchain", "I love salad", "my car", "a dog"]
# 创建集群并添加文本
cluster = Bagel.from_texts(cluster_name="testing", texts=texts)
```


```python
# 相似性搜索
cluster.similarity_search("bagel", k=3)
```



```output
[Document(page_content='hello bagel', metadata={}),
 Document(page_content='my car', metadata={}),
 Document(page_content='I love salad', metadata={})]
```



```python
# 分数是距离度量，因此越低越好
cluster.similarity_search_with_score("bagel", k=3)
```



```output
[(Document(page_content='hello bagel', metadata={}), 0.27392977476119995),
 (Document(page_content='my car', metadata={}), 1.4783176183700562),
 (Document(page_content='I love salad', metadata={}), 1.5342965126037598)]
```



```python
# 删除集群
cluster.delete_cluster()
```

## 从文档创建 VectorStore


```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)[:10]
```


```python
# 使用文档创建集群
cluster = Bagel.from_documents(cluster_name="testing_with_docs", documents=docs)
```


```python
# 相似性搜索
query = "总统对 Ketanji Brown Jackson 说了什么"
docs = cluster.similarity_search(query)
print(docs[0].page_content[:102])
```
```output
Madam Speaker, Madam Vice President, our First Lady and Second Gentleman. Members of Congress and the
```

## 从集群中获取所有文本/文档


```python
texts = ["hello bagel", "this is langchain"]
cluster = Bagel.from_texts(cluster_name="testing", texts=texts)
cluster_data = cluster.get()
```


```python
# 所有键
cluster_data.keys()
```



```output
dict_keys(['ids', 'embeddings', 'metadatas', 'documents'])
```



```python
# 所有值和键
cluster_data
```



```output
{'ids': ['578c6d24-3763-11ee-a8ab-b7b7b34f99ba',
  '578c6d25-3763-11ee-a8ab-b7b7b34f99ba',
  'fb2fc7d8-3762-11ee-a8ab-b7b7b34f99ba',
  'fb2fc7d9-3762-11ee-a8ab-b7b7b34f99ba',
  '6b40881a-3762-11ee-a8ab-b7b7b34f99ba',
  '6b40881b-3762-11ee-a8ab-b7b7b34f99ba',
  '581e691e-3762-11ee-a8ab-b7b7b34f99ba',
  '581e691f-3762-11ee-a8ab-b7b7b34f99ba'],
 'embeddings': None,
 'metadatas': [{}, {}, {}, {}, {}, {}, {}, {}],
 'documents': ['hello bagel',
  'this is langchain',
  'hello bagel',
  'this is langchain',
  'hello bagel',
  'this is langchain',
  'hello bagel',
  'this is langchain']}
```



```python
cluster.delete_cluster()
```

## 使用元数据创建集群并通过元数据过滤


```python
texts = ["hello bagel", "this is langchain"]
metadatas = [{"source": "notion"}, {"source": "google"}]

cluster = Bagel.from_texts(cluster_name="testing", texts=texts, metadatas=metadatas)
cluster.similarity_search_with_score("hello bagel", where={"source": "notion"})
```



```output
[(Document(page_content='hello bagel', metadata={'source': 'notion'}), 0.0)]
```



```python
# 删除集群
cluster.delete_cluster()
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)