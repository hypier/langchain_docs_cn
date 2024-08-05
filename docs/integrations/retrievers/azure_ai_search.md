---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/azure_ai_search.ipynb
sidebar_label: Azure AI 搜索
---

# AzureAISearchRetriever

## 概述
[Azure AI Search](https://learn.microsoft.com/azure/search/search-what-is-azure-search)（前称为 `Azure Cognitive Search`）是微软的云搜索服务，为开发者提供基础设施、API 和工具，以便在大规模下进行向量、关键词和混合查询的信息检索。

`AzureAISearchRetriever` 是一个集成模块，可以从非结构化查询中返回文档。它基于 BaseRetriever 类，并且针对 2023-11-01 稳定的 Azure AI Search REST API 版本，这意味着它支持向量索引和查询。

本指南将帮助您开始使用 Azure AI Search [检索器](/docs/concepts/#retrievers)。有关所有 `AzureAISearchRetriever` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.azure_ai_search.AzureAISearchRetriever.html)。

`AzureAISearchRetriever` 替代了即将被弃用的 `AzureCognitiveSearchRetriever`。我们建议您切换到基于最新稳定版本搜索 API 的新版本。

### 集成细节

| 检索器 | 自托管 | 云服务 | 包 |
| :--- | :--- | :---: | :---: |
[AzureAISearchRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.azure_ai_search.AzureAISearchRetriever.html) | ❌ | ✅ | langchain_community |

## 设置

要使用此模块，您需要：

+ 一个 Azure AI Search 服务。如果您注册 Azure 试用版，可以[免费创建一个](https://learn.microsoft.com/azure/search/search-create-service-portal)。免费服务的配额较低，但足以运行本笔记本中的代码。

+ 一个包含向量字段的现有索引。可以通过多种方式创建索引，包括使用[向量存储模块](../vectorstores/azuresearch.md)。或者，[尝试 Azure AI Search REST API](https://learn.microsoft.com/azure/search/search-get-started-vector)。

+ 一个 API 密钥。创建搜索服务时会生成 API 密钥。如果您只是查询一个索引，可以使用查询 API 密钥，否则请使用管理员 API 密钥。有关详细信息，请参见[查找您的 API 密钥](https://learn.microsoft.com/azure/search/search-security-api-keys?tabs=rest-use%2Cportal-find%2Cportal-query#find-existing-keys)。

然后，我们可以将搜索服务名称、索引名称和 API 密钥设置为环境变量（或者，您可以将它们作为参数传递给 `AzureAISearchRetriever`）。搜索索引提供可搜索的内容。


```python
import os

os.environ["AZURE_AI_SEARCH_SERVICE_NAME"] = "<YOUR_SEARCH_SERVICE_NAME>"
os.environ["AZURE_AI_SEARCH_INDEX_NAME"] = "<YOUR_SEARCH_INDEX_NAME>"
os.environ["AZURE_AI_SEARCH_API_KEY"] = "<YOUR_API_KEY>"
```

如果您希望从单个查询中获取自动跟踪，您还可以通过取消注释以下内容来设置您的[LangSmith](https://docs.smith.langchain.com/) API 密钥：


```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### 安装

这个检索器位于 `langchain-community` 包中。我们还需要一些额外的依赖：

```python
%pip install --upgrade --quiet langchain-community
%pip install --upgrade --quiet langchain-openai
%pip install --upgrade --quiet  azure-search-documents>=11.4
%pip install --upgrade --quiet  azure-identity
```

## 实例化

对于 `AzureAISearchRetriever`，提供 `index_name`、`content_key` 和设置 `top_k` 为您希望检索的结果数量。将 `top_k` 设置为零（默认值）将返回所有结果。

```python
from langchain_community.retrievers import AzureAISearchRetriever

retriever = AzureAISearchRetriever(
    content_key="content", top_k=1, index_name="langchain-vector-demo"
)
```

## 用法

现在您可以使用它从 Azure AI Search 检索文档。 
这是您将要调用的方法。它将返回与查询相关的所有文档。 


```python
retriever.invoke("here is my unstructured query string")
```

## 示例

本节演示如何在内置示例数据上使用检索器。如果您已经在搜索服务上拥有向量索引，可以跳过此步骤。

首先提供端点和密钥。由于我们在此步骤中创建向量索引，请指定一个文本嵌入模型以获取文本的向量表示。此示例假设使用 Azure OpenAI，并部署了 text-embedding-ada-002。因为此步骤创建索引，请务必使用搜索服务的管理员 API 密钥。

```python
import os

from langchain_community.document_loaders import DirectoryLoader, TextLoader
from langchain_community.retrievers import AzureAISearchRetriever
from langchain_community.vectorstores import AzureSearch
from langchain_openai import AzureOpenAIEmbeddings, OpenAIEmbeddings
from langchain_text_splitters import TokenTextSplitter

os.environ["AZURE_AI_SEARCH_SERVICE_NAME"] = "<YOUR_SEARCH_SERVICE_NAME>"
os.environ["AZURE_AI_SEARCH_INDEX_NAME"] = "langchain-vector-demo"
os.environ["AZURE_AI_SEARCH_API_KEY"] = "<YOUR_SEARCH_SERVICE_ADMIN_API_KEY>"
azure_endpoint: str = "<YOUR_AZURE_OPENAI_ENDPOINT>"
azure_openai_api_key: str = "<YOUR_AZURE_OPENAI_API_KEY>"
azure_openai_api_version: str = "2023-05-15"
azure_deployment: str = "text-embedding-ada-002"
```

我们将使用 Azure OpenAI 的嵌入模型将文档转换为存储在 Azure AI Search 向量存储中的嵌入。我们还将索引名称设置为 `langchain-vector-demo`。这将创建一个与该索引名称相关联的新向量存储。

```python
embeddings = AzureOpenAIEmbeddings(
    model=azure_deployment,
    azure_endpoint=azure_endpoint,
    openai_api_key=azure_openai_api_key,
)

vector_store: AzureSearch = AzureSearch(
    embedding_function=embeddings.embed_query,
    azure_search_endpoint=os.getenv("AZURE_AI_SEARCH_SERVICE_NAME"),
    azure_search_key=os.getenv("AZURE_AI_SEARCH_API_KEY"),
    index_name="langchain-vector-demo",
)
```

接下来，我们将数据加载到新创建的向量存储中。对于此示例，我们加载 `state_of_the_union.txt` 文件。我们将文本按 400 个标记的块进行分割，并且没有重叠。最后，文档作为嵌入添加到我们的向量存储中。

```python
from langchain_community.document_loaders import TextLoader
from langchain_text_splitters import CharacterTextSplitter

loader = TextLoader("../../how_to/state_of_the_union.txt", encoding="utf-8")

documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=400, chunk_overlap=0)
docs = text_splitter.split_documents(documents)

vector_store.add_documents(documents=docs)
```

接下来，我们将创建一个检索器。当前的 `index_name` 变量是来自上一步的 `langchain-vector-demo`。如果您跳过了向量存储的创建，请在参数中提供您的索引名称。在此查询中，将返回顶部结果。

```python
retriever = AzureAISearchRetriever(
    content_key="content", top_k=1, index_name="langchain-vector-demo"
)
```

现在我们可以从上传的文档中检索与查询相关的数据。

```python
retriever.invoke("does the president have a plan for covid-19?")
```

## 在链中使用


```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the context provided.

Context: {context}

Question: {question}"""
)

llm = ChatOpenAI(model="gpt-3.5-turbo-0125")


def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)


chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
```


```python
chain.invoke("does the president have a plan for covid-19?")
```

## API 参考

有关所有 `AzureAISearchRetriever` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.azure_ai_search.AzureAISearchRetriever.html)。

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)