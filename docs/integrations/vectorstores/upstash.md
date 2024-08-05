---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/upstash.ipynb
---

# Upstash Vector

> [Upstash Vector](https://upstash.com/docs/vector/overall/whatisvector) 是一个无服务器的向量数据库，旨在处理向量嵌入。
>
> 向量 langchain 集成是对 [upstash-vector](https://github.com/upstash/vector-py) 包的封装。
>
> 该 Python 包在后台使用 [vector rest api](https://upstash.com/docs/vector/api/get-started)。

## 安装

从 [upstash 控制台](https://console.upstash.com/vector) 创建一个免费的向量数据库，设置所需的维度和距离度量。

然后可以通过以下方式创建 `UpstashVectorStore` 实例：

- 提供环境变量 `UPSTASH_VECTOR_URL` 和 `UPSTASH_VECTOR_TOKEN`

- 将它们作为参数传递给构造函数

- 将 Upstash Vector `Index` 实例传递给构造函数

此外，还需要一个 `Embeddings` 实例将给定文本转换为嵌入。在这里我们使用 `OpenAIEmbeddings` 作为示例

```python
%pip install langchain-openai langchain langchain-community upstash-vector
```

```python
import os

from langchain_community.vectorstores.upstash import UpstashVectorStore
from langchain_openai import OpenAIEmbeddings

os.environ["OPENAI_API_KEY"] = "<YOUR_OPENAI_KEY>"
os.environ["UPSTASH_VECTOR_REST_URL"] = "<YOUR_UPSTASH_VECTOR_URL>"
os.environ["UPSTASH_VECTOR_REST_TOKEN"] = "<YOUR_UPSTASH_VECTOR_TOKEN>"

# Create an embeddings instance
embeddings = OpenAIEmbeddings()

# Create a vector store instance
store = UpstashVectorStore(embedding=embeddings)
```

创建 `UpstashVectorStore` 的另一种方法是 [通过选择模型创建 Upstash Vector 索引](https://upstash.com/docs/vector/features/embeddingmodels#using-a-model)，并传递 `embedding=True`。在这种配置下，文档或查询将作为文本发送到 Upstash，并在那里进行嵌入。

```python
store = UpstashVectorStore(embedding=True)
```

如果您有兴趣尝试这种方法，可以像上面那样更新 `store` 的初始化，并运行其余的教程。

## 加载文档

加载一个示例文本文件并将其拆分为可以转换为向量嵌入的块。


```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

docs[:3]
```

## 插入文档

vectorstore 使用嵌入对象嵌入文本块，并将其批量插入数据库。这将返回插入向量的 ID 数组。

```python
inserted_vectors = store.add_documents(docs)

inserted_vectors[:5]
```



```output
['82b3781b-817c-4a4d-8f8b-cbd07c1d005a',
 'a20e0a49-29d8-465e-8eae-0bc5ac3d24dc',
 'c19f4108-b652-4890-873e-d4cad00f1b1a',
 '23d1fcf9-6ee1-4638-8c70-0f5030762301',
 '2d775784-825d-4627-97a3-fee4539d8f58']
```


store


```python
store.add_texts(
    [
        "A timeless tale set in the Jazz Age, this novel delves into the lives of affluent socialites, their pursuits of wealth, love, and the elusive American Dream. Amidst extravagant parties and glittering opulence, the story unravels the complexities of desire, ambition, and the consequences of obsession.",
        "Set in a small Southern town during the 1930s, this novel explores themes of racial injustice, moral growth, and empathy through the eyes of a young girl. It follows her father, a principled lawyer, as he defends a black man accused of assaulting a white woman, confronting deep-seated prejudices and challenging societal norms along the way.",
        "A chilling portrayal of a totalitarian regime, this dystopian novel offers a bleak vision of a future world dominated by surveillance, propaganda, and thought control. Through the eyes of a disillusioned protagonist, it explores the dangers of totalitarianism and the erosion of individual freedom in a society ruled by fear and oppression.",
        "Set in the English countryside during the early 19th century, this novel follows the lives of the Bennet sisters as they navigate the intricate social hierarchy of their time. Focusing on themes of marriage, class, and societal expectations, the story offers a witty and insightful commentary on the complexities of romantic relationships and the pursuit of happiness.",
        "Narrated by a disillusioned teenager, this novel follows his journey of self-discovery and rebellion against the phoniness of the adult world. Through a series of encounters and reflections, it explores themes of alienation, identity, and the search for authenticity in a society marked by conformity and hypocrisy.",
        "In a society where emotion is suppressed and individuality is forbidden, one man dares to defy the oppressive regime. Through acts of rebellion and forbidden love, he discovers the power of human connection and the importance of free will.",
        "Set in a future world devastated by environmental collapse, this novel follows a group of survivors as they struggle to survive in a harsh, unforgiving landscape. Amidst scarcity and desperation, they must confront moral dilemmas and question the nature of humanity itself.",
    ],
    [
        {"title": "The Great Gatsby", "author": "F. Scott Fitzgerald", "year": 1925},
        {"title": "To Kill a Mockingbird", "author": "Harper Lee", "year": 1960},
        {"title": "1984", "author": "George Orwell", "year": 1949},
        {"title": "Pride and Prejudice", "author": "Jane Austen", "year": 1813},
        {"title": "The Catcher in the Rye", "author": "J.D. Salinger", "year": 1951},
        {"title": "Brave New World", "author": "Aldous Huxley", "year": 1932},
        {"title": "The Road", "author": "Cormac McCarthy", "year": 2006},
    ],
)
```



```output
['fe1f7a7b-42e2-4828-88b0-5b449c49fe86',
 '154a0021-a99c-427e-befb-f0b2b18ed83c',
 'a8218226-18a9-4ab5-ade5-5a71b19a7831',
 '62b7ef97-83bf-4b6d-8c93-f471796244dc',
 'ab43fd2e-13df-46d4-8cf7-e6e16506e4bb',
 '6841e7f9-adaa-41d9-af3d-0813ee52443f',
 '45dda5a1-f0c1-4ac7-9acb-50253e4ee493']
```

## 查询

数据库可以使用向量或文本提示进行查询。
如果使用文本提示，它会首先被转换为嵌入，然后进行查询。

`k` 参数指定要从查询中返回多少结果。


```python
result = store.similarity_search("The United States of America", k=5)
result
```





```python
result = store.similarity_search("dystopia", k=3, filter="year < 2000")
result
```

## 使用分数查询

查询的分数可以包含在每个结果中。

> 查询请求中返回的分数是一个介于 0 和 1 之间的标准化值，其中 1 表示最高相似度，0 表示最低相似度，无论使用何种相似度函数。有关更多信息，请查看 [docs](https://upstash.com/docs/vector/overall/features#vector-similarity-functions)。

```python
result = store.similarity_search_with_score("The United States of America", k=5)

for doc, score in result:
    print(f"{doc.metadata} - {score}")
```
```output
{'source': '../../how_to/state_of_the_union.txt'} - 0.87391514
{'source': '../../how_to/state_of_the_union.txt'} - 0.8549463
{'source': '../../how_to/state_of_the_union.txt'} - 0.847913
{'source': '../../how_to/state_of_the_union.txt'} - 0.84328896
{'source': '../../how_to/state_of_the_union.txt'} - 0.832347
```

## 删除向量

可以通过其 ID 删除向量

```python
store.delete(inserted_vectors)
```

## 清空向量数据库

这将清空向量数据库


```python
store.delete(delete_all=True)
```

## 获取有关向量数据库的信息

您可以使用 info 函数获取有关数据库的信息，例如距离度量维度。

> 当插入发生时，数据库会进行索引。在此期间，无法查询新的向量。 `pendingVectorCount` 表示当前正在被索引的向量数量。

```python
store.info()
```

```output
InfoResult(vector_count=42, pending_vector_count=0, index_size=6470, dimension=384, similarity_function='COSINE')
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)