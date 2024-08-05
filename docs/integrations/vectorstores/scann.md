---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/scann.ipynb
---

# ScaNN

ScaNN（可扩展最近邻）是一种用于大规模高效向量相似性搜索的方法。

ScaNN 包括用于最大内积搜索的搜索空间剪枝和量化，并且支持其他距离函数，例如欧几里得距离。该实现经过优化，适用于支持 AVX2 的 x86 处理器。有关更多详细信息，请参阅其 [Google Research github](https://github.com/google-research/google-research/tree/master/scann)。

您需要使用 `pip install -qU langchain-community` 安装 `langchain-community` 才能使用此集成。

## 安装
通过 pip 安装 ScaNN。或者，您可以按照 [ScaNN 网站](https://github.com/google-research/google-research/tree/master/scann#building-from-source) 上的说明从源代码安装。

```python
%pip install --upgrade --quiet  scann
```

## 检索演示

以下是如何将 ScaNN 与 Huggingface Embeddings 结合使用的示例。

```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import ScaNN
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)


embeddings = HuggingFaceEmbeddings()

db = ScaNN.from_documents(docs, embeddings)
query = "What did the president say about Ketanji Brown Jackson"
docs = db.similarity_search(query)

docs[0]
```

## RetrievalQA 演示

接下来，我们演示如何将 ScaNN 与 Google PaLM API 结合使用。

您可以从 https://developers.generativeai.google/tutorials/setup 获取 API 密钥。


```python
from langchain.chains import RetrievalQA
from langchain_community.chat_models.google_palm import ChatGooglePalm

palm_client = ChatGooglePalm(google_api_key="YOUR_GOOGLE_PALM_API_KEY")

qa = RetrievalQA.from_chain_type(
    llm=palm_client,
    chain_type="stuff",
    retriever=db.as_retriever(search_kwargs={"k": 10}),
)
```


```python
print(qa.run("What did the president say about Ketanji Brown Jackson?"))
```
```output
The president said that Ketanji Brown Jackson is one of our nation's top legal minds, who will continue Justice Breyer's legacy of excellence.
```

```python
print(qa.run("What did the president say about Michael Phelps?"))
```
```output
The president did not mention Michael Phelps in his speech.
```

## 保存和加载本地检索索引


```python
db.save_local("/tmp/db", "state_of_union")
restored_db = ScaNN.load_local("/tmp/db", embeddings, index_name="state_of_union")
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)