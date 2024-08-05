---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/momento_vector_index.ipynb
---

# Momento Vector Index (MVI)

>[MVI](https://gomomento.com)：最具生产力、最易于使用的无服务器向量索引，适用于您的数据。要开始使用 MVI，只需注册一个帐户。无需处理基础设施、管理服务器或担心扩展。MVI 是一项能够自动扩展以满足您需求的服务。

要注册并访问 MVI，请访问 [Momento Console](https://console.gomomento.com)。

# 设置

## 安装前提条件

您需要：
- [`momento`](https://pypi.org/project/momento/) 包以与 MVI 进行交互，以及
- openai 包以与 OpenAI API 进行交互。
- tiktoken 包以进行文本分词。

```python
%pip install --upgrade --quiet  momento langchain-openai langchain-community tiktoken
```

## 输入 API 密钥


```python
import getpass
import os
```

### Momento: 数据索引

访问 [Momento 控制台](https://console.gomomento.com) 获取您的 API 密钥。


```python
os.environ["MOMENTO_API_KEY"] = getpass.getpass("Momento API Key:")
```

### OpenAI：用于文本嵌入


```python
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```

# 加载数据

这里我们使用来自 Langchain 的示例数据集，即国情咨文。

首先，我们加载相关模块：

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import MomentoVectorIndex
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

然后我们加载数据：

```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
len(documents)
```



```output
1
```


注意数据是一个大文件，因此只有一个文档：

```python
len(documents[0].page_content)
```



```output
38539
```


由于这是一个大文本文件，我们将其拆分成多个块以便于问答。这样，用户的问题将从最相关的块中得到回答。

```python
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)
len(docs)
```



```output
42
```

# 索引您的数据

索引您的数据就像实例化 `MomentoVectorIndex` 对象一样简单。在这里，我们使用 `from_documents` 辅助函数来实例化并索引数据：


```python
vector_db = MomentoVectorIndex.from_documents(
    docs, OpenAIEmbeddings(), index_name="sotu"
)
```

这将使用您的 API 密钥连接到 Momento Vector Index 服务并索引数据。如果索引之前不存在，这个过程会为您创建它。数据现在可以被搜索。

# 查询您的数据

## 直接针对索引提问

查询数据的最直接方法是针对索引进行搜索。我们可以使用 `VectorStore` API 如下操作：

```python
query = "What did the president say about Ketanji Brown Jackson"
docs = vector_db.similarity_search(query)
```

```python
docs[0].page_content
```

```output
'Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.'
```

虽然这包含了关于 Ketanji Brown Jackson 的相关信息，但我们没有一个简洁、易读的答案。我们将在下一部分解决这个问题。

## 使用 LLM 生成流畅的回答

通过在 MVI 中索引的数据，我们可以与任何利用向量相似性搜索的链进行集成。在这里，我们使用 `RetrievalQA` 链来演示如何从索引数据中回答问题。

首先，我们加载相关模块：

```python
from langchain.chains import RetrievalQA
from langchain_openai import ChatOpenAI
```

然后，我们实例化检索 QA 链：

```python
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
qa_chain = RetrievalQA.from_chain_type(llm, retriever=vector_db.as_retriever())
```

```python
qa_chain({"query": "What did the president say about Ketanji Brown Jackson?"})
```

```output
{'query': 'What did the president say about Ketanji Brown Jackson?',
 'result': "The President said that he nominated Circuit Court of Appeals Judge Ketanji Brown Jackson to serve on the United States Supreme Court. He described her as one of the nation's top legal minds and mentioned that she has received broad support from various groups, including the Fraternal Order of Police and former judges appointed by Democrats and Republicans."}
```

# 下一步

就这样！您现在已经索引了数据，并可以使用 Momento Vector Index 查询它。您可以使用相同的索引从任何支持向量相似性搜索的链中查询数据。

使用 Momento，您不仅可以索引向量数据，还可以缓存 API 调用并存储聊天消息历史记录。查看其他 Momento langchain 集成以了解更多信息。

要了解有关 Momento Vector Index 的更多信息，请访问 [Momento 文档](https://docs.gomomento.com)。

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)