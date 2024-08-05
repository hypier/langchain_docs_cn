---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/vectorstores/pinecone.ipynb
---

# Pinecone

>[Pinecone](https://docs.pinecone.io/docs/overview) 是一个功能广泛的向量数据库。

本笔记本展示了如何使用与 `Pinecone` 向量数据库相关的功能。

设置以下环境变量以便在本文档中进行操作：
- `OPENAI_API_KEY`：您的 OpenAI API 密钥，用于使用 `OpenAIEmbeddings`


```python
%pip install --upgrade --quiet  \
    langchain-pinecone \
    langchain-openai \
    langchain \
    langchain-community \
    pinecone-notebooks
```

迁移说明：如果您正在从 `langchain_community.vectorstores` 的 Pinecone 实现迁移，您可能需要在安装 `langchain-pinecone` 之前移除 `pinecone-client` v2 依赖项，因为它依赖于 `pinecone-client` v3。

首先，让我们将国情咨文文档分割成块 `docs`。


```python
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

embeddings = OpenAIEmbeddings()
```

现在让我们创建一个新的 Pinecone 账户，或登录到您现有的账户，并创建一个 API 密钥以在本笔记本中使用。


```python
from pinecone_notebooks.colab import Authenticate

Authenticate()
```

新创建的 API 密钥已存储在 `PINECONE_API_KEY` 环境变量中。我们将使用它来设置 Pinecone 客户端。


```python
import os

pinecone_api_key = os.environ.get("PINECONE_API_KEY")
pinecone_api_key

import time

from pinecone import Pinecone, ServerlessSpec

pc = Pinecone(api_key=pinecone_api_key)
```

接下来，让我们连接到您的 Pinecone 索引。如果名为 `index_name` 的索引不存在，将会创建一个。


```python
import time

index_name = "langchain-index"  # 如有需要请更改

existing_indexes = [index_info["name"] for index_info in pc.list_indexes()]

if index_name not in existing_indexes:
    pc.create_index(
        name=index_name,
        dimension=1536,
        metric="cosine",
        spec=ServerlessSpec(cloud="aws", region="us-east-1"),
    )
    while not pc.describe_index(index_name).status["ready"]:
        time.sleep(1)

index = pc.Index(index_name)
```

现在我们的 Pinecone 索引已设置完毕，我们可以使用 `PineconeVectorStore.from_documents` 将这些分块文档作为内容插入。


```python
from langchain_pinecone import PineconeVectorStore

docsearch = PineconeVectorStore.from_documents(docs, embeddings, index_name=index_name)
```


```python
query = "What did the president say about Ketanji Brown Jackson"
docs = docsearch.similarity_search(query)
print(docs[0].page_content)
```
```output
今晚。我呼吁参议院：通过《投票自由法案》。通过《约翰·刘易斯投票权法案》。同时，请通过《披露法案》，让美国人知道谁在资助我们的选举。

今晚，我想表彰一位为这个国家奉献一生的人：史蒂芬·布雷耶法官——一位退伍军人、宪法学者，以及即将退休的美国最高法院法官。布雷耶法官，感谢您的服务。

总统最重要的宪法责任之一就是提名某人担任美国最高法院法官。

而我在四天前做到了这一点，当时我提名了巡回上诉法院法官凯坦吉·布朗·杰克逊。她是我们国家顶尖的法律人才之一，将继续布雷耶法官卓越的遗产。
```

### 向现有索引添加更多文本

可以使用 `add_texts` 函数将更多文本嵌入并更新到现有的 Pinecone 索引中



```python
vectorstore = PineconeVectorStore(index_name=index_name, embedding=embeddings)

vectorstore.add_texts(["More text!"])
```



```output
['24631802-4bad-44a7-a4ba-fd71f00cc160']
```

### 最大边际相关性搜索

除了在检索器对象中使用相似性搜索外，您还可以将 `mmr` 作为检索器使用。



```python
retriever = docsearch.as_retriever(search_type="mmr")
matched_docs = retriever.invoke(query)
for i, d in enumerate(matched_docs):
    print(f"\n## Document {i}\n")
    print(d.page_content)
```

或者直接使用 `max_marginal_relevance_search`：


```python
found_docs = docsearch.max_marginal_relevance_search(query, k=2, fetch_k=10)
for i, doc in enumerate(found_docs):
    print(f"{i + 1}.", doc.page_content, "\n")
```

## 相关

- 向量存储 [概念指南](/docs/concepts/#vector-stores)
- 向量存储 [操作指南](/docs/how_to/#vector-stores)