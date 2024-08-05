---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/typesense.ipynb
---

# Typesense

> [Typesense](https://typesense.org) 是一个开源的内存搜索引擎，您可以选择 [自托管](https://typesense.org/docs/guide/install-typesense#option-2-local-machine-self-hosting) 或在 [Typesense Cloud](https://cloud.typesense.org/) 上运行。
>
> Typesense 通过将整个索引存储在 RAM 中（并在磁盘上备份）来专注于性能，同时还通过简化可用选项和设置良好的默认值来提供开箱即用的开发者体验。
>
> 它还允许您将基于属性的过滤与向量查询结合起来，以获取最相关的文档。

本笔记本将向您展示如何将 Typesense 用作您的 VectorStore。

让我们首先安装所需的依赖项：


```python
%pip install --upgrade --quiet  typesense openapi-schema-pydantic langchain-openai langchain-community tiktoken
```

我们想使用 `OpenAIEmbeddings`，所以我们需要获取 OpenAI API 密钥。


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
```


```python
from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import Typesense
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter
```

让我们导入我们的测试数据集：


```python
loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```


```python
docsearch = Typesense.from_documents(
    docs,
    embeddings,
    typesense_client_params={
        "host": "localhost",  # Use xxx.a1.typesense.net for Typesense Cloud
        "port": "8108",  # Use 443 for Typesense Cloud
        "protocol": "http",  # Use https for Typesense Cloud
        "typesense_api_key": "xyz",
        "typesense_collection_name": "lang-chain",
    },
)
```

## 相似性搜索


```python
query = "What did the president say about Ketanji Brown Jackson"
found_docs = docsearch.similarity_search(query)
```


```python
print(found_docs[0].page_content)
```

## Typesense 作为检索器

Typesense 和其他所有向量存储一样，是一个 LangChain 检索器，通过使用余弦相似度。

```python
retriever = docsearch.as_retriever()
retriever
```

```python
query = "What did the president say about Ketanji Brown Jackson"
retriever.invoke(query)[0]
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)