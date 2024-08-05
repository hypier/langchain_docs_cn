---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/singlestoredb.ipynb
---

# SingleStoreDB

>[SingleStoreDB](https://singlestore.com/) 是一个高性能的分布式 SQL 数据库，支持在 [云](https://www.singlestore.com/cloud/) 和本地部署。它提供向量存储和向量函数，包括 [dot_product](https://docs.singlestore.com/managed-service/en/reference/sql-reference/vector-functions/dot_product.html) 和 [euclidean_distance](https://docs.singlestore.com/managed-service/en/reference/sql-reference/vector-functions/euclidean_distance.html)，从而支持需要文本相似性匹配的 AI 应用程序。

此笔记本展示了如何使用一个使用 `SingleStoreDB` 的检索器。

```python
# Establishing a connection to the database is facilitated through the singlestoredb Python connector.
# Please ensure that this connector is installed in your working environment.
%pip install --upgrade --quiet  singlestoredb
```

## 从向量存储创建检索器


```python
import getpass
import os

# 我们想使用 OpenAIEmbeddings，因此我们必须获取 OpenAI API 密钥。
os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")

from langchain_community.document_loaders import TextLoader
from langchain_community.vectorstores import SingleStoreDB
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()

# 将连接 URL 设置为环境变量
os.environ["SINGLESTOREDB_URL"] = "root:pass@localhost:3306/db"

# 将文档加载到存储中
docsearch = SingleStoreDB.from_documents(
    docs,
    embeddings,
    table_name="notebook",  # 使用自定义名称的表
)

# 从向量存储创建检索器
retriever = docsearch.as_retriever(search_kwargs={"k": 2})
```

## 使用检索器搜索


```python
result = retriever.invoke("What did the president say about Ketanji Brown Jackson")
print(docs[0].page_content)
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)