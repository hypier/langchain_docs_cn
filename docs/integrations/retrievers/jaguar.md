---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/jaguar.ipynb
---

# JaguarDB 向量数据库

>[JaguarDB 向量数据库](http://www.jaguardb.com/windex.html
>
>1. 这是一个分布式向量数据库
>2. JaguarDB 的“ZeroMove”功能实现即时水平扩展
>3. 多模态：嵌入、文本、图像、视频、PDF、音频、时间序列和地理空间
>4. 全主模式：允许并行读取和写入
>5. 异常检测能力
>6. RAG 支持：将 LLM 与专有和实时数据结合
>7. 共享元数据：在多个向量索引之间共享元数据
>8. 距离度量：欧几里得、余弦、内积、曼哈顿、切比雪夫、汉明、杰卡德、闵可夫斯基

## 前提条件

运行此文件中的示例有两个要求。
1. 您必须安装并设置JaguarDB服务器及其HTTP网关服务器。
   请参阅以下说明：
   [www.jaguardb.com](http://www.jaguardb.com)

2. 您必须为JaguarDB安装http客户端包：
   ```
       pip install -U jaguardb-http-client
   ```

## RAG 与 Langchain

本节演示如何在 langchain 软件栈中与 LLM 及 Jaguar 进行对话。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores.jaguar import Jaguar
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

""" 
将文本文件加载到一组文档中 
"""
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=300)
docs = text_splitter.split_documents(documents)

"""
实例化一个 Jaguar 向量存储
"""
### Jaguar HTTP 端点
url = "http://192.168.5.88:8080/fwww/"

### 使用 OpenAI 嵌入模型
embeddings = OpenAIEmbeddings()

### Pod 是向量的数据库
pod = "vdb"

### 向量存储名称
store = "langchain_rag_store"

### 向量索引名称
vector_index = "v"

### 向量索引的类型
# cosine: 距离度量
# fraction: 嵌入向量为十进制数
# float: 使用浮点数存储的值
vector_type = "cosine_fraction_float"

### 每个嵌入向量的维度
vector_dimension = 1536

### 实例化一个 Jaguar 存储对象
vectorstore = Jaguar(
    pod, store, vector_index, vector_type, vector_dimension, url, embeddings
)

"""
必须进行登录以授权客户端。
环境变量 JAGUAR_API_KEY 或文件 $HOME/.jagrc
应包含访问 JaguarDB 服务器的 API 密钥。
"""
vectorstore.login()


"""
在 JaguarDB 数据库服务器上创建向量存储。
这应该只执行一次。
"""
# 向量存储的额外元数据字段
metadata = "category char(16)"

# 存储的文本字段字符数
text_size = 4096

#  在服务器上创建一个向量存储
vectorstore.create(metadata, text_size)

"""
将文本分割器中的文本添加到我们的向量存储中
"""
vectorstore.add_documents(docs)

""" 获取检索器对象 """
retriever = vectorstore.as_retriever()
# retriever = vectorstore.as_retriever(search_kwargs={"where": "m1='123' and m2='abc'"})

""" 检索器对象可以与 LangChain 和 LLM 一起使用 """
```

## 与 Jaguar 向量存储的交互

用户可以直接与 Jaguar 向量存储进行相似性搜索和异常检测。



```python
from langchain_community.vectorstores.jaguar import Jaguar
from langchain_openai import OpenAIEmbeddings

# Instantiate a Jaguar vector store object
url = "http://192.168.3.88:8080/fwww/"
pod = "vdb"
store = "langchain_test_store"
vector_index = "v"
vector_type = "cosine_fraction_float"
vector_dimension = 10
embeddings = OpenAIEmbeddings()
vectorstore = Jaguar(
    pod, store, vector_index, vector_type, vector_dimension, url, embeddings
)

# Login for authorization
vectorstore.login()

# Create the vector store with two metadata fields
# This needs to be run only once.
metadata_str = "author char(32), category char(16)"
vectorstore.create(metadata_str, 1024)

# Add a list of texts
texts = ["foo", "bar", "baz"]
metadatas = [
    {"author": "Adam", "category": "Music"},
    {"author": "Eve", "category": "Music"},
    {"author": "John", "category": "History"},
]
ids = vectorstore.add_texts(texts=texts, metadatas=metadatas)

#  Search similar text
output = vectorstore.similarity_search(
    query="foo",
    k=1,
    metadatas=["author", "category"],
)
assert output[0].page_content == "foo"
assert output[0].metadata["author"] == "Adam"
assert output[0].metadata["category"] == "Music"
assert len(output) == 1

# Search with filtering (where)
where = "author='Eve'"
output = vectorstore.similarity_search(
    query="foo",
    k=3,
    fetch_k=9,
    where=where,
    metadatas=["author", "category"],
)
assert output[0].page_content == "bar"
assert output[0].metadata["author"] == "Eve"
assert output[0].metadata["category"] == "Music"
assert len(output) == 1

# Anomaly detection
result = vectorstore.is_anomalous(
    query="dogs can jump high",
)
assert result is False

# Remove all data in the store
vectorstore.clear()
assert vectorstore.count() == 0

# Remove the store completely
vectorstore.drop()

# Logout
vectorstore.logout()
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)