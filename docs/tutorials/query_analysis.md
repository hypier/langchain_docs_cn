---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/tutorials/query_analysis.ipynb
sidebar_position: 0
---

# 构建查询分析系统

:::info 前提条件

本指南假设您熟悉以下概念：

- [文档加载器](/docs/concepts/#document-loaders)
- [聊天模型](/docs/concepts/#chat-models)
- [嵌入](/docs/concepts/#embedding-models)
- [向量存储](/docs/concepts/#vector-stores)
- [检索](/docs/concepts/#retrieval)

:::

本页面将展示如何在一个基本的端到端示例中使用查询分析。这将包括创建一个简单的搜索引擎，展示在将原始用户问题传递给该搜索时发生的失败模式，以及查询分析如何帮助解决该问题的示例。有许多不同的查询分析技术，而这个端到端示例不会展示所有这些技术。

为了这个示例，我们将对LangChain YouTube视频进行检索。

## 设置
#### 安装依赖


```python
# %pip install -qU langchain langchain-community langchain-openai youtube-transcript-api pytube langchain-chroma
```

#### 设置环境变量

在本示例中，我们将使用 OpenAI：


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()

# 可选，取消注释以使用 LangSmith 跟踪运行。请在此处注册： https://smith.langchain.com.
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

### 加载文档

我们可以使用 `YouTubeLoader` 来加载一些 LangChain 视频的转录文本：

```python
from langchain_community.document_loaders import YoutubeLoader

urls = [
    "https://www.youtube.com/watch?v=HAn9vnJy6S4",
    "https://www.youtube.com/watch?v=dA1cHGACXCo",
    "https://www.youtube.com/watch?v=ZcEMLz27sL4",
    "https://www.youtube.com/watch?v=hvAPnpSfSGo",
    "https://www.youtube.com/watch?v=EhlPDL4QrWY",
    "https://www.youtube.com/watch?v=mmBo8nlu2j0",
    "https://www.youtube.com/watch?v=rQdibOsL1ps",
    "https://www.youtube.com/watch?v=28lC4fqukoc",
    "https://www.youtube.com/watch?v=es-9MgxB-uc",
    "https://www.youtube.com/watch?v=wLRHwKuKvOE",
    "https://www.youtube.com/watch?v=ObIltMaRJvY",
    "https://www.youtube.com/watch?v=DjuXACWYkkU",
    "https://www.youtube.com/watch?v=o7C9ld6Ln-M",
]
docs = []
for url in urls:
    docs.extend(YoutubeLoader.from_youtube_url(url, add_video_info=True).load())
```

```python
import datetime

# 添加一些额外的元数据：视频发布的年份
for doc in docs:
    doc.metadata["publish_year"] = int(
        datetime.datetime.strptime(
            doc.metadata["publish_date"], "%Y-%m-%d %H:%M:%S"
        ).strftime("%Y")
    )
```

以下是我们加载的视频的标题：

```python
[doc.metadata["title"] for doc in docs]
```

```output
['OpenGPTs',
 '构建一个网络 RAG 聊天机器人：使用 LangChain、Exa（前称 Metaphor）、LangSmith 和 Hosted Langserve',
 '流式事件：介绍新的 `stream_events` 方法',
 'LangGraph：多智能体工作流',
 '使用 Pinecone Serverless 构建和部署 RAG 应用',
 '自动提示生成器（与 Hosted LangServe 一起）',
 '使用 TypeScript 构建全栈 RAG 应用',
 '开始使用多模态 LLMs',
 'SQL 研究助手',
 '思维骨架：从零开始构建新模板',
 '对 LangChain 文档进行 RAG 基准测试',
 '从零开始构建研究助手',
 'LangServe 和 LangChain 模板网络研讨会']
```

这是与每个视频相关的元数据。我们可以看到每个文档还有标题、观看次数、发布日期和时长：

```python
docs[0].metadata
```

```output
{'source': 'HAn9vnJy6S4',
 'title': 'OpenGPTs',
 'description': '未知',
 'view_count': 7210,
 'thumbnail_url': 'https://i.ytimg.com/vi/HAn9vnJy6S4/hq720.jpg',
 'publish_date': '2024-01-31 00:00:00',
 'length': 1530,
 'author': 'LangChain',
 'publish_year': 2024}
```

这是文档内容的一个示例：

```python
docs[0].page_content[:500]
```

```output
"hello today I want to talk about open gpts open gpts is a project that we built here at linkchain uh that replicates the GPT store in a few ways so it creates uh end user-facing friendly interface to create different Bots and these Bots can have access to different tools and they can uh be given files to retrieve things over and basically it's a way to create a variety of bots and expose the configuration of these Bots to end users it's all open source um it can be used with open AI it can be us"
```

### 索引文档

每当我们执行检索时，我们需要创建一个可以查询的文档索引。我们将使用向量存储来索引我们的文档，并且我们会先对它们进行分块，以使我们的检索更加简洁和精确：

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(chunk_size=2000)
chunked_docs = text_splitter.split_documents(docs)
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Chroma.from_documents(
    chunked_docs,
    embeddings,
)
```

## 无需查询分析的检索

我们可以直接对用户问题执行相似性搜索，以查找与问题相关的内容：

```python
search_results = vectorstore.similarity_search("how do I build a RAG agent")
print(search_results[0].metadata["title"])
print(search_results[0].page_content[:500])
```
```output
Build and Deploy a RAG app with Pinecone Serverless
hi this is Lance from the Lang chain team and today we're going to be building and deploying a rag app using pine con serval list from scratch so we're going to kind of walk through all the code required to do this and I'll use these slides as kind of a guide to kind of lay the the ground work um so first what is rag so under capoy has this pretty nice visualization that shows LMS as a kernel of a new kind of operating system and of course one of the core components of our operating system is th
```
这效果很好！我们的第一个结果与问题相当相关。

如果我们想要从特定时间段搜索结果呢？

```python
search_results = vectorstore.similarity_search("videos on RAG published in 2023")
print(search_results[0].metadata["title"])
print(search_results[0].metadata["publish_date"])
print(search_results[0].page_content[:500])
```
```output
OpenGPTs
2024-01-31
hardcoded that it will always do a retrieval step here the assistant decides whether to do a retrieval step or not sometimes this is good sometimes this is bad sometimes it you don't need to do a retrieval step when I said hi it didn't need to call it tool um but other times you know the the llm might mess up and not realize that it needs to do a retrieval step and so the rag bot will always do a retrieval step so it's more focused there because this is also a simpler architecture so it's always
```
我们的第一个结果来自2024年（尽管我们请求的是2023年的视频），而且与输入的相关性不高。由于我们只是对文档内容进行搜索，因此无法根据任何文档属性过滤结果。

这只是可能出现的一种失败模式。现在让我们看看基本的查询分析形式如何解决这个问题！

## 查询分析

我们可以使用查询分析来改善检索结果。这将涉及定义一个包含一些日期过滤器的 **查询模式**，并使用函数调用模型将用户问题转换为结构化查询。

### 查询模式
在这种情况下，我们将为发布日期设置明确的最小和最大属性，以便进行过滤。


```python
from typing import Optional

from langchain_core.pydantic_v1 import BaseModel, Field


class Search(BaseModel):
    """在关于软件库的教程视频数据库中进行搜索。"""

    query: str = Field(
        ...,
        description="应用于视频转录的相似性搜索查询。",
    )
    publish_year: Optional[int] = Field(None, description="视频发布的年份")
```

### 查询生成

为了将用户问题转换为结构化查询，我们将利用 OpenAI 的工具调用 API。具体来说，我们将使用新的 [ChatModel.with_structured_output()](/docs/how_to/structured_output) 构造函数来处理将模式传递给模型并解析输出。

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI

system = """You are an expert at converting user questions into database queries. \
You have access to a database of tutorial videos about a software library for building LLM-powered applications. \
Given a question, return a list of database queries optimized to retrieve the most relevant results.

If there are acronyms or words you are not familiar with, do not try to rephrase them."""
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        ("human", "{question}"),
    ]
)
llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
structured_llm = llm.with_structured_output(Search)
query_analyzer = {"question": RunnablePassthrough()} | prompt | structured_llm
```
```output
/Users/bagatur/langchain/libs/core/langchain_core/_api/beta_decorator.py:86: LangChainBetaWarning: The function `with_structured_output` is in beta. It is actively being worked on, so the API may change.
  warn_beta(
```
让我们看看我们的分析器为我们之前搜索的问题生成了什么查询：

```python
query_analyzer.invoke("how do I build a RAG agent")
```

```output
Search(query='build RAG agent', publish_year=None)
```

```python
query_analyzer.invoke("videos on RAG published in 2023")
```

```output
Search(query='RAG', publish_year=2023)
```

## 查询分析的检索

我们的查询分析看起来不错；现在让我们尝试使用生成的查询来实际执行检索。

**注意：** 在我们的示例中，我们指定了 `tool_choice="Search"`。这将强制 LLM 调用一个 - 仅一个 - 工具，这意味着我们将始终有一个优化的查询进行查找。请注意，这并不总是如此 - 有关如何处理未返回或返回多个优化查询的情况，请参见其他指南。

```python
from typing import List

from langchain_core.documents import Document
```

```python
def retrieval(search: Search) -> List[Document]:
    if search.publish_year is not None:
        # 这是特定于 Chroma 的语法，
        # 我们正在使用的向量数据库。
        _filter = {"publish_year": {"$eq": search.publish_year}}
    else:
        _filter = None
    return vectorstore.similarity_search(search.query, filter=_filter)
```

```python
retrieval_chain = query_analyzer | retrieval
```

我们现在可以在之前的问题输入上运行这个链，并看到它仅返回该年份的结果！

```python
results = retrieval_chain.invoke("RAG tutorial published in 2023")
```

```python
[(doc.metadata["title"], doc.metadata["publish_date"]) for doc in results]
```

```output
[('Getting Started with Multi-Modal LLMs', '2023-12-20 00:00:00'),
 ('LangServe and LangChain Templates Webinar', '2023-11-02 00:00:00'),
 ('Getting Started with Multi-Modal LLMs', '2023-12-20 00:00:00'),
 ('Building a Research Assistant from Scratch', '2023-11-16 00:00:00')]
```