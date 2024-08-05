---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/chroma.ipynb
---

# Chroma

>[Chroma](https://docs.trychroma.com/getting-started) 是一个以AI为原生的开源向量数据库，专注于开发者的生产力和幸福感。Chroma 采用 Apache 2.0 许可证。

使用以下命令安装 Chroma：

```sh
pip install langchain-chroma
```

Chroma 可以在多种模式下运行。以下是与 LangChain 集成的每种模式的示例。
- `in-memory` - 在 Python 脚本或 Jupyter Notebook 中
- `in-memory with persistance` - 在脚本或 Notebook 中并保存/加载到磁盘
- `in a docker container` - 作为在本地机器或云中运行的服务器

像其他数据库一样，您可以：
- `.add` 
- `.get` 
- `.update`
- `.upsert`
- `.delete`
- `.peek`
- 和 `.query` 进行相似性搜索。

查看完整文档请访问 [docs](https://docs.trychroma.com/reference/py-collection)。要直接访问这些方法，您可以使用 `._collection.method()`

## 基本示例

在这个基本示例中，我们获取最近的国情咨文，将其拆分成多个部分，使用开源嵌入模型进行嵌入，将其加载到 Chroma 中，然后进行查询。

```python
# import
from langchain_chroma import Chroma
from langchain_community.document_loaders import TextLoader
from langchain_community.embeddings.sentence_transformer import (
    SentenceTransformerEmbeddings,
)
from langchain_text_splitters import CharacterTextSplitter

# load the document and split it into chunks
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()

# split it into chunks
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

# create the open-source embedding function
embedding_function = SentenceTransformerEmbeddings(model_name="all-MiniLM-L6-v2")

# load it into Chroma
db = Chroma.from_documents(docs, embedding_function)

# query it
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)

# print results
print(docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

## 基本示例（包括保存到磁盘）

扩展前面的示例，如果您想保存到磁盘，只需初始化 Chroma 客户端并传递要保存数据的目录。

`注意`：Chroma 尽力自动将数据保存到磁盘，然而多个内存客户端可能会相互干扰。作为最佳实践，在任何时候每个路径上只运行一个客户端。

```python
# save to disk
db2 = Chroma.from_documents(docs, embedding_function, persist_directory="./chroma_db")
docs = db2.similarity_search(query)

# load from disk
db3 = Chroma(persist_directory="./chroma_db", embedding_function=embedding_function)
docs = db3.similarity_search(query)
print(docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

## 将 Chroma 客户端传递给 Langchain

您还可以创建一个 Chroma 客户端并将其传递给 LangChain。这在您希望更轻松地访问底层数据库时特别有用。

您还可以指定希望 LangChain 使用的集合名称。


```python
import chromadb

persistent_client = chromadb.PersistentClient()
collection = persistent_client.get_or_create_collection("collection_name")
collection.add(ids=["1", "2", "3"], documents=["a", "b", "c"])

langchain_chroma = Chroma(
    client=persistent_client,
    collection_name="collection_name",
    embedding_function=embedding_function,
)

print("There are", langchain_chroma._collection.count(), "in the collection")
```
```output
Add of existing embedding ID: 1
Add of existing embedding ID: 2
Add of existing embedding ID: 3
Add of existing embedding ID: 1
Add of existing embedding ID: 2
Add of existing embedding ID: 3
Add of existing embedding ID: 1
Insert of existing embedding ID: 1
Add of existing embedding ID: 2
Insert of existing embedding ID: 2
Add of existing embedding ID: 3
Insert of existing embedding ID: 3
``````output
There are 3 in the collection
```

## 基本示例（使用 Docker 容器）

您也可以在 Docker 容器中单独运行 Chroma 服务器，创建一个客户端与之连接，然后将其传递给 LangChain。

Chroma 能够处理多个 `Collections` 的文档，但 LangChain 接口期望只有一个，因此我们需要指定集合名称。LangChain 使用的默认集合名称是 "langchain"。

以下是如何克隆、构建和运行 Docker 镜像：
```sh
git clone git@github.com:chroma-core/chroma.git
```

编辑 `docker-compose.yml` 文件，并在 `environment` 下添加 `ALLOW_RESET=TRUE`
```yaml
    ...
    command: uvicorn chromadb.app:app --reload --workers 1 --host 0.0.0.0 --port 8000 --log-config log_config.yml
    environment:
      - IS_PERSISTENT=TRUE
      - ALLOW_RESET=TRUE
    ports:
      - 8000:8000
    ...
```

然后运行 `docker-compose up -d --build`


```python
# create the chroma client
import uuid

import chromadb
from chromadb.config import Settings

client = chromadb.HttpClient(settings=Settings(allow_reset=True))
client.reset()  # resets the database
collection = client.create_collection("my_collection")
for doc in docs:
    collection.add(
        ids=[str(uuid.uuid1())], metadatas=doc.metadata, documents=doc.page_content
    )

# tell LangChain to use our client and collection name
db4 = Chroma(
    client=client,
    collection_name="my_collection",
    embedding_function=embedding_function,
)
query = "What did the president say about Ketanji Brown Jackson"
docs = db4.similarity_search(query)
print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯投票权法案》。同时，通过《披露法案》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：大法官斯蒂芬·布雷耶——一位退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，感谢您的服务。

总统最严肃的宪法责任之一是提名某人担任美国最高法院法官。

而我在 4 天前就做了这一提名，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官卓越的遗产。
```

## 更新和删除

在构建真实应用程序时，您希望不仅添加数据，还要更新和删除数据。

Chroma 让用户提供 `ids` 以简化这里的记账。`ids` 可以是文件名，或者是组合哈希，例如 `filename_paragraphNumber` 等等。

Chroma 支持所有这些操作——尽管其中一些仍在通过 LangChain 接口进行集成。额外的工作流程改进将很快添加。

以下是一个基本示例，展示如何执行各种操作：

```python
# create simple ids
ids = [str(i) for i in range(1, len(docs) + 1)]

# add data
example_db = Chroma.from_documents(docs, embedding_function, ids=ids)
docs = example_db.similarity_search(query)
print(docs[0].metadata)

# update the metadata for a document
docs[0].metadata = {
    "source": "../../how_to/state_of_the_union.txt",
    "new_value": "hello world",
}
example_db.update_document(ids[0], docs[0])
print(example_db._collection.get(ids=[ids[0]]))

# delete the last document
print("count before", example_db._collection.count())
example_db._collection.delete(ids=[ids[-1]])
print("count after", example_db._collection.count())
```
```output
{'source': '../../../state_of_the_union.txt'}
{'ids': ['1'], 'embeddings': None, 'metadatas': [{'new_value': 'hello world', 'source': '../../../state_of_the_union.txt'}], 'documents': ['Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.']}
count before 46
count after 45
```

## 使用 OpenAI 嵌入

许多人喜欢使用 OpenAIEmbeddings，以下是如何设置它。

```python
# 获取令牌: https://platform.openai.com/account/api-keys

from getpass import getpass

from langchain_openai import OpenAIEmbeddings

OPENAI_API_KEY = getpass()
```

```python
import os

os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

```python
embeddings = OpenAIEmbeddings()
new_client = chromadb.EphemeralClient()
openai_lc_client = Chroma.from_documents(
    docs, embeddings, client=new_client, collection_name="openai_collection"
)

query = "总统对 Ketanji Brown Jackson 说了什么"
docs = openai_lc_client.similarity_search(query)
print(docs[0].page_content)
```
```output
Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. 

Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. 

One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. 

And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

## 其他信息

### 相似度搜索与得分

返回的距离得分是余弦距离。因此，得分越低越好。


```python
docs = db.similarity_search_with_score(query)
```


```python
docs[0]
```



```output
(Document(page_content='今晚。我呼吁参议院：通过投票自由法案。通过约翰·刘易斯投票权法案。在此期间，通过披露法案，让美国人知道谁在资助我们的选举。\n\n今晚，我想向一位为国家奉献一生的人致敬：大法官斯蒂芬·布雷耶——一位陆军退伍军人、宪法学者，以及即将退休的美国最高法院大法官。布雷耶大法官，谢谢您的服务。\n\n总统最重要的宪法责任之一就是提名某人担任美国最高法院大法官。\n\n我在4天前做了这件事，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律人才之一，将继续布雷耶大法官卓越的遗产。', metadata={'source': '../../../state_of_the_union.txt'}),
 1.1972057819366455)
```

### 检索器选项

本节讨论如何将 Chroma 用作检索器的不同选项。

#### MMR

除了在检索器对象中使用相似性搜索外，您还可以使用 `mmr`。

```python
retriever = db.as_retriever(search_type="mmr")
```

```python
retriever.invoke(query)[0]
```

```output
Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': '../../../state_of_the_union.txt'})
```

### 基于元数据的过滤

在处理集合之前，缩小范围可能会很有帮助。

例如，可以使用 get 方法根据元数据过滤集合。


```python
# filter collection for updated source
example_db.get(where={"source": "some_other_source"})
```



```output
{'ids': [], 'embeddings': None, 'metadatas': [], 'documents': []}
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)