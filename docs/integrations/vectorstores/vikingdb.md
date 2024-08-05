---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/vikingdb.ipynb
---

# viking DB

>[viking DB](https://www.volcengine.com/docs/6459/1163946) 是一个存储、索引和管理由深度神经网络和其他机器学习（ML）模型生成的大规模嵌入向量的数据库。

本笔记本展示了如何使用与 VikingDB 向量数据库相关的功能。

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 以使用此集成。

要运行，您应该有一个 [viking DB 实例正在运行](https://www.volcengine.com/docs/6459/1165058)。

```python
!pip install --upgrade volcengine
```

我们想使用 VikingDBEmbeddings，因此我们必须获取 VikingDB API 密钥。

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.vikingdb import VikingDB, VikingDBConfig
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
```

```python
loader = TextLoader("./test.txt")
documents = loader.load()
text_splitter = RecursiveCharacterTextSplitter(chunk_size=10, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```

```python
db = VikingDB.from_documents(
    docs,
    embeddings,
    connection_args=VikingDBConfig(
        host="host", region="region", ak="ak", sk="sk", scheme="http"
    ),
    drop_old=True,
)
```

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)
```

```python
docs[0].page_content
```

### 使用 Viking DB Collections 对数据进行分区

您可以在同一个 Viking DB 实例中将不同的无关文档存储在不同的集合中，以保持上下文。

以下是创建新集合的方法


```python
db = VikingDB.from_documents(
    docs,
    embeddings,
    connection_args=VikingDBConfig(
        host="host", region="region", ak="ak", sk="sk", scheme="http"
    ),
    collection_name="collection_1",
    drop_old=True,
)
```

以下是检索已存储集合的方法


```python
db = VikingDB.from_documents(
    embeddings,
    connection_args=VikingDBConfig(
        host="host", region="region", ak="ak", sk="sk", scheme="http"
    ),
    collection_name="collection_1",
)
```

检索后，您可以像往常一样对其进行查询。

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)