---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/google_vertex_ai_search.ipynb
sidebar_label: Google Vertex AI 搜索
---

# Google Vertex AI 搜索

## 概述

>[Google Vertex AI Search](https://cloud.google.com/enterprise-search)（前称为 `Enterprise Search` 在 `Generative AI App Builder` 上）是 `Google Cloud` 提供的 [Vertex AI](https://cloud.google.com/vertex-ai) 机器学习平台的一部分。
>
>`Vertex AI Search` 使组织能够快速为客户和员工构建基于生成式 AI 的搜索引擎。它基于多种 `Google Search` 技术，包括语义搜索，通过使用自然语言处理和机器学习技术推断内容之间的关系和用户查询输入的意图，从而提供比传统基于关键字的搜索技术更相关的结果。Vertex AI Search 还受益于 Google 在理解用户搜索方式方面的专业知识，并考虑内容相关性来排序显示的结果。

>`Vertex AI Search` 可通过 `Google Cloud Console` 和 API 进行企业工作流集成。

本笔记本演示了如何配置 `Vertex AI Search` 并使用 Vertex AI Search [检索器](/docs/concepts/#retrievers)。Vertex AI Search 检索器封装了 [Python 客户端库](https://cloud.google.com/generative-ai-app-builder/docs/libraries#client-libraries-install-python)，并使用它访问 [Search Service API](https://cloud.google.com/python/docs/reference/discoveryengine/latest/google.cloud.discoveryengine_v1beta.services.search_service)。

有关所有 `VertexAISearchRetriever` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/vertex_ai_search/langchain_google_community.vertex_ai_search.VertexAISearchRetriever.html)。

### 集成详情

| 检索器 | 自托管 | 云服务 | 包 |
| :--- | :--- | :---: | :---: |
[VertexAISearchRetriever](https://api.python.langchain.com/en/latest/vertex_ai_search/langchain_google_community.vertex_ai_search.VertexAISearchRetriever.html) | ❌ | ✅ | langchain_google_community |

## 设置

### 安装

您需要安装 `langchain-google-community` 和 `google-cloud-discoveryengine` 包才能使用 Vertex AI Search 检索器。

```python
%pip install -qU langchain-google-community google-cloud-discoveryengine
```

### 配置对 Google Cloud 和 Vertex AI Search 的访问

从 2023 年 8 月起，Vertex AI Search 已经可以在没有允许列表的情况下使用。

在使用检索器之前，您需要完成以下步骤：

#### 创建搜索引擎并填充非结构化数据存储

- 按照 [Vertex AI Search 入门指南](https://cloud.google.com/generative-ai-app-builder/docs/try-enterprise-search) 中的说明设置 Google Cloud 项目和 Vertex AI Search。
- [使用 Google Cloud 控制台创建非结构化数据存储](https://cloud.google.com/generative-ai-app-builder/docs/create-engine-es#unstructured-data)
  - 使用 `gs://cloud-samples-data/gen-app-builder/search/alphabet-investor-pdfs` Cloud Storage 文件夹中的示例 PDF 文档填充它。
  - 确保使用 `Cloud Storage (without metadata)` 选项。

#### 设置凭据以访问 Vertex AI Search API

[Vertex AI Search 客户端库](https://cloud.google.com/generative-ai-app-builder/docs/libraries) 由 Vertex AI Search 检索器使用，提供对 Google Cloud 进行程序化身份验证的高级语言支持。
客户端库支持 [应用程序默认凭据 (ADC)](https://cloud.google.com/docs/authentication/application-default-credentials)；库会在一组定义的位置查找凭据，并使用这些凭据对 API 请求进行身份验证。
使用 ADC，您可以在多种环境中（例如本地开发或生产）将凭据提供给您的应用程序，而无需修改应用程序代码。

如果在 [Google Colab](https://colab.google) 中运行，请使用 `google.colab.google.auth` 进行身份验证，否则请遵循 [支持的方法](https://cloud.google.com/docs/authentication/application-default-credentials) 之一，以确保您的应用程序默认凭据已正确设置。

```python
import sys

if "google.colab" in sys.modules:
    from google.colab import auth as google_auth

    google_auth.authenticate_user()
```

### 配置和使用 Vertex AI 搜索检索器

Vertex AI 搜索检索器在 `langchain_google_community.VertexAISearchRetriever` 类中实现。`get_relevant_documents` 方法返回一个 `langchain.schema.Document` 文档列表，其中每个文档的 `page_content` 字段填充了文档内容。根据在 Vertex AI 搜索中使用的数据类型（网站、结构化或非结构化），`page_content` 字段的填充方式如下：

- 具有高级索引的网站：与查询匹配的 `extractive answer`。`metadata` 字段填充了从中提取段落或答案的文档的元数据（如果有）。
- 非结构化数据源：与查询匹配的 `extractive segment` 或 `extractive answer`。`metadata` 字段填充了从中提取段落或答案的文档的元数据（如果有）。
- 结构化数据源：包含从结构化数据源返回的所有字段的字符串 JSON。`metadata` 字段填充了文档的元数据（如果有）。

#### 提取答案与提取段落

提取答案是与每个搜索结果一起返回的逐字文本。它直接从原始文档中提取。提取答案通常显示在网页的顶部，以便为最终用户提供与其查询相关的简要答案。提取答案适用于网站和非结构化搜索。

提取段落是与每个搜索结果一起返回的逐字文本。提取段落通常比提取答案更冗长。提取段落可以作为查询的答案显示，并可用于执行后处理任务以及作为大型语言模型生成答案或新文本的输入。提取段落适用于非结构化搜索。

有关提取段落和提取答案的更多信息，请参阅 [产品文档](https://cloud.google.com/generative-ai-app-builder/docs/snippets)。

注意：提取段落需要启用 [企业版](https://cloud.google.com/generative-ai-app-builder/docs/about-advanced-features#enterprise-features) 功能。

创建检索器实例时，可以指定多个参数，以控制访问哪个数据存储以及如何处理自然语言查询，包括提取答案和段落的配置。

#### 必填参数：

- `project_id` - 您的 Google Cloud 项目 ID。
- `location_id` - 数据存储的位置。
  - `global`（默认）
  - `us`
  - `eu`

以下之一：
- `search_engine_id` - 您要使用的搜索应用程序的 ID。（对于混合搜索必需）
- `data_store_id` - 您要使用的数据存储的 ID。

`project_id`、`search_engine_id` 和 `data_store_id` 参数可以在检索器的构造函数中显式提供，也可以通过环境变量 - `PROJECT_ID`、`SEARCH_ENGINE_ID` 和 `DATA_STORE_ID` 提供。

您还可以配置多个可选参数，包括：

- `max_documents` - 用于提供提取段落或提取答案的最大文档数。
- `get_extractive_answers` - 默认情况下，检索器配置为返回提取段落。
  - 将此字段设置为 `True` 以返回提取答案。仅在 `engine_data_type` 设置为 `0`（非结构化）时使用。
- `max_extractive_answer_count` - 每个搜索结果返回的最大提取答案数。
  - 最多返回 5 个答案。仅在 `engine_data_type` 设置为 `0`（非结构化）时使用。
- `max_extractive_segment_count` - 每个搜索结果返回的最大提取段落数。
  - 目前将返回一个段落。仅在 `engine_data_type` 设置为 `0`（非结构化）时使用。
- `filter` - 基于数据存储中文档相关的元数据的搜索结果过滤表达式。
- `query_expansion_condition` - 指定在何种条件下进行查询扩展。
  - `0` - 未指定的查询扩展条件。在这种情况下，服务器行为默认为禁用。
  - `1` - 禁用查询扩展。仅使用精确的搜索查询，即使 SearchResponse.total_size 为零。
  - `2` - 由搜索 API 自动构建的查询扩展。
- `engine_data_type` - 定义 Vertex AI 搜索数据类型
  - `0` - 非结构化数据
  - `1` - 结构化数据
  - `2` - 网站数据
  - `3` - [混合搜索](https://cloud.google.com/generative-ai-app-builder/docs/create-data-store-es#multi-data-stores)

### `GoogleCloudEnterpriseSearchRetriever` 的迁移指南

在之前的版本中，此检索器被称为 `GoogleCloudEnterpriseSearchRetriever`。

要更新到新的检索器，请进行以下更改：

- 将导入更改为： `from langchain.retrievers import GoogleCloudEnterpriseSearchRetriever` -> `from langchain_google_community import VertexAISearchRetriever`。
- 将所有类引用更改为 `GoogleCloudEnterpriseSearchRetriever` -> `VertexAISearchRetriever`。

注意：使用检索器时，如果您希望从单个查询中获取自动跟踪，您还可以通过取消注释以下内容来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：


```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

## 实例化

### 配置并使用检索器处理**非结构化**数据与抽取段落


```python
from langchain_google_community import (
    VertexAIMultiTurnSearchRetriever,
    VertexAISearchRetriever,
)

PROJECT_ID = "<YOUR PROJECT ID>"  # 设置为您的项目ID
LOCATION_ID = "<YOUR LOCATION>"  # 设置为您的数据存储位置
SEARCH_ENGINE_ID = "<YOUR SEARCH APP ID>"  # 设置为您的搜索应用ID
DATA_STORE_ID = "<YOUR DATA STORE ID>"  # 设置为您的数据存储ID
```


```python
retriever = VertexAISearchRetriever(
    project_id=PROJECT_ID,
    location_id=LOCATION_ID,
    data_store_id=DATA_STORE_ID,
    max_documents=3,
)
```


```python
query = "What are Alphabet's Other Bets?"

result = retriever.invoke(query)
for doc in result:
    print(doc)
```

### 配置并使用检索器获取**非结构化**数据的抽取式答案



```python
retriever = VertexAISearchRetriever(
    project_id=PROJECT_ID,
    location_id=LOCATION_ID,
    data_store_id=DATA_STORE_ID,
    max_documents=3,
    max_extractive_answer_count=3,
    get_extractive_answers=True,
)

result = retriever.invoke(query)
for doc in result:
    print(doc)
```

### 配置和使用检索器以获取**结构化**数据



```python
retriever = VertexAISearchRetriever(
    project_id=PROJECT_ID,
    location_id=LOCATION_ID,
    data_store_id=DATA_STORE_ID,
    max_documents=3,
    engine_data_type=1,
)

result = retriever.invoke(query)
for doc in result:
    print(doc)
```

### 配置并使用检索器获取**网站**数据，采用高级网站索引

```python
retriever = VertexAISearchRetriever(
    project_id=PROJECT_ID,
    location_id=LOCATION_ID,
    data_store_id=DATA_STORE_ID,
    max_documents=3,
    max_extractive_answer_count=3,
    get_extractive_answers=True,
    engine_data_type=2,
)

result = retriever.invoke(query)
for doc in result:
    print(doc)
```

### 配置和使用检索器以获取**混合**数据



```python
retriever = VertexAISearchRetriever(
    project_id=PROJECT_ID,
    location_id=LOCATION_ID,
    search_engine_id=SEARCH_ENGINE_ID,
    max_documents=3,
    engine_data_type=3,
)

result = retriever.invoke(query)
for doc in result:
    print(doc)
```

### 配置和使用多轮搜索的检索器

[带后续的搜索](https://cloud.google.com/generative-ai-app-builder/docs/multi-turn-search) 基于生成式 AI 模型，与常规非结构化数据搜索不同。



```python
retriever = VertexAIMultiTurnSearchRetriever(
    project_id=PROJECT_ID, location_id=LOCATION_ID, data_store_id=DATA_STORE_ID
)

result = retriever.invoke(query)
for doc in result:
    print(doc)
```

## 用法

根据上述示例，我们使用 `.invoke` 发出单个查询。因为检索器是 Runnables，我们也可以使用 [Runnable interface](/docs/concepts/#runnable-interface) 中的任何方法，例如 `.batch`。

## 在链中使用

我们还可以将检索器纳入[链](/docs/how_to/sequence/)中，以构建更大的应用程序，例如一个简单的[RAG](/docs/tutorials/rag/)应用程序。出于演示目的，我们还实例化了一个VertexAI聊天模型。有关设置说明，请参阅相应的Vertex [集成文档](/docs/integrations/chat/google_vertex_ai_palm/)。

```python
%pip install -qU langchain-google-vertexai
```

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_google_vertexai import ChatVertexAI

prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the context provided.

Context: {context}

Question: {question}"""
)

llm = ChatVertexAI(model_name="chat-bison", temperature=0)


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
chain.invoke(query)
```

## API 参考

有关所有 `VertexAISearchRetriever` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/vertex_ai_search/langchain_google_community.vertex_ai_search.VertexAISearchRetriever.html)。

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)