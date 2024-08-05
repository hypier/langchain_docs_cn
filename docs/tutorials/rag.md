---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/tutorials/rag.ipynb
---

# 构建一个检索增强生成 (RAG) 应用

由 LLMs 驱动的最强大应用之一是复杂的问答 (Q&A) 聊天机器人。这些应用可以回答有关特定源信息的问题。这些应用使用一种称为检索增强生成的技术，或称 RAG。

本教程将展示如何构建一个简单的 Q&A 应用程序，基于文本数据源。在此过程中，我们将讨论一个典型的 Q&A 架构，并强调更多高级 Q&A 技术的额外资源。我们还将看到 LangSmith 如何帮助我们追踪和理解我们的应用程序。随着应用程序复杂性的增加，LangSmith 将变得越来越有用。

如果您已经熟悉基本的检索，您可能还会对这篇 [不同检索技术的高级概述](/docs/concepts/#retrieval) 感兴趣。

## 什么是 RAG？

RAG 是一种通过额外数据增强 LLM 知识的技术。

LLM 可以推理各种主题，但它们的知识仅限于训练时所用的公共数据，且截至特定时间点。如果您希望构建能够推理私有数据或在模型截止日期后引入的数据的 AI 应用程序，您需要用模型所需的特定信息来增强模型的知识。将适当的信息引入并插入到模型提示中的过程称为检索增强生成（RAG）。

LangChain 有多个组件旨在帮助构建问答应用程序，以及更广泛的 RAG 应用程序。

**注意**：在这里我们重点关注非结构化数据的问答。如果您对结构化数据上的 RAG 感兴趣，请查看我们关于 [SQL 数据的问答教程](/docs/tutorials/sql_qa)。

## 概念
一个典型的 RAG 应用程序有两个主要组件：

**索引**：一个从源获取数据并进行索引的管道。*这通常在离线进行。*

**检索和生成**：实际的 RAG 链，在运行时接收用户查询并从索引中检索相关数据，然后将其传递给模型。

从原始数据到答案的最常见完整序列如下：

### 索引
1. **加载**: 首先我们需要加载我们的数据。这是通过 [Document Loaders](/docs/concepts/#document-loaders) 完成的。
2. **拆分**: [Text splitters](/docs/concepts/#text-splitters) 将大型 `Documents` 拆分为较小的块。这对于索引数据和将其传递给模型都很有用，因为大型块更难搜索，并且无法适应模型的有限上下文窗口。
3. **存储**: 我们需要一个地方来存储和索引我们的拆分，以便后续可以进行搜索。这通常使用 [VectorStore](/docs/concepts/#vector-stores) 和 [Embeddings](/docs/concepts/#embedding-models) 模型来完成。

![index_diagram](../../static/img/rag_indexing.png)

### 检索与生成
4. **检索**：根据用户输入，从存储中使用 [Retriever](/docs/concepts/#retrievers) 检索相关的分片。
5. **生成**：一个 [ChatModel](/docs/concepts/#chat-models) / [LLM](/docs/concepts/#llms) 使用包含问题和检索数据的提示生成答案。

![retrieval_diagram](../../static/img/rag_retrieval_generation.png)

## 设置

### Jupyter Notebook

本指南（以及文档中的大多数其他指南）使用 [Jupyter notebooks](https://jupyter.org/) 并假设读者也使用它。Jupyter notebooks 非常适合学习如何使用 LLM 系统，因为在很多情况下可能会出现问题（意外输出、API 崩溃等），在交互环境中逐步进行指南是更好理解它们的好方法。

本教程和其他教程最方便的运行方式是使用 Jupyter notebook。有关安装的说明，请参见 [这里](https://jupyter.org/install)。

### 安装

本教程需要以下 langchain 依赖：

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CodeBlock from "@theme/CodeBlock";

<Tabs>
  <TabItem value="pip" label="Pip" default>
    <CodeBlock language="bash">pip install langchain langchain_community langchain_chroma</CodeBlock>
  </TabItem>
  <TabItem value="conda" label="Conda">
    <CodeBlock language="bash">conda install langchain langchain_community langchain_chroma -c conda-forge</CodeBlock>
  </TabItem>
</Tabs>

有关更多详细信息，请参阅我们的 [安装指南](/docs/how_to/installation)。

### LangSmith

您使用 LangChain 构建的许多应用程序将包含多个步骤和多次调用 LLM。  
随着这些应用程序变得越来越复杂，能够检查您的链或代理内部究竟发生了什么变得至关重要。  
做到这一点的最佳方法是使用 [LangSmith](https://smith.langchain.com)。

在您注册上述链接后，请确保设置您的环境变量以开始记录追踪：

```shell
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="..."
```

或者，如果在笔记本中，您可以通过以下方式设置它们：

```python
import getpass
import os

os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 预览

在本指南中，我们将构建一个应用程序，用于回答有关网站内容的问题。我们将使用的具体网站是Lilian Weng的[LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)博客文章，该文章允许我们询问有关内容的问题。

我们可以创建一个简单的索引管道和RAG链来实现这一点，代码大约为20行：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />


```python
import bs4
from langchain import hub
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

# Load, chunk and index the contents of the blog.
loader = WebBaseLoader(
    web_paths=("https://lilianweng.github.io/posts/2023-06-23-agent/",),
    bs_kwargs=dict(
        parse_only=bs4.SoupStrainer(
            class_=("post-content", "post-title", "post-header")
        )
    ),
)
docs = loader.load()

text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
splits = text_splitter.split_documents(docs)
vectorstore = Chroma.from_documents(documents=splits, embedding=OpenAIEmbeddings())

# Retrieve and generate using the relevant snippets of the blog.
retriever = vectorstore.as_retriever()
prompt = hub.pull("rlm/rag-prompt")


def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)


rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

rag_chain.invoke("What is Task Decomposition?")
```



```output
'Task Decomposition is a process where a complex task is broken down into smaller, simpler steps or subtasks. This technique is utilized to enhance model performance on complex tasks by making them more manageable. It can be done by using language models with simple prompting, task-specific instructions, or with human inputs.'
```



```python
# cleanup
vectorstore.delete_collection()
```

查看[LangSmith trace](https://smith.langchain.com/public/1c6ca97e-445b-4d00-84b4-c7befcbc59fe/r)。

## 详细步骤讲解

让我们一步一步地分析上述代码，以真正理解发生了什么。

## 1. 索引：加载 {#indexing-load}

我们需要首先加载博客文章的内容。我们可以使用
[DocumentLoaders](/docs/concepts#document-loaders)
来实现，这些对象从源加载数据并返回一个
[Documents](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html)的列表。
`Document`是一个包含一些`page_content`（str）和`metadata`（dict）的对象。

在这种情况下，我们将使用
[WebBaseLoader](/docs/integrations/document_loaders/web_base)，
它使用`urllib`从网页 URL 加载 HTML，并使用`BeautifulSoup`将其解析为文本。我们可以通过`bs_kwargs`将参数传递给`BeautifulSoup`解析器，从而自定义 HTML -\> 文本解析（参见
[BeautifulSoup
docs](https://beautiful-soup-4.readthedocs.io/en/latest/#beautifulsoup)）。
在这种情况下，只有类名为“post-content”、“post-title”或“post-header”的 HTML 标签是相关的，因此我们将删除所有其他标签。


```python
import bs4
from langchain_community.document_loaders import WebBaseLoader

# Only keep post title, headers, and content from the full HTML.
bs4_strainer = bs4.SoupStrainer(class_=("post-title", "post-header", "post-content"))
loader = WebBaseLoader(
    web_paths=("https://lilianweng.github.io/posts/2023-06-23-agent/",),
    bs_kwargs={"parse_only": bs4_strainer},
)
docs = loader.load()

len(docs[0].page_content)
```



```output
43131
```



```python
print(docs[0].page_content[:500])
```
```output


      LLM Powered Autonomous Agents
    
Date: June 23, 2023  |  Estimated Reading Time: 31 min  |  Author: Lilian Weng


Building agents with LLM (large language model) as its core controller is a cool concept. Several proof-of-concepts demos, such as AutoGPT, GPT-Engineer and BabyAGI, serve as inspiring examples. The potentiality of LLM extends beyond generating well-written copies, stories, essays and programs; it can be framed as a powerful general problem solver.
Agent System Overview#
In
```

### 深入了解

`DocumentLoader`: 从源加载数据作为 `Documents` 列表的对象。

- [文档](/docs/how_to#document-loaders):
  关于如何使用 `DocumentLoaders` 的详细文档。
- [集成](/docs/integrations/document_loaders/): 160+
  种可供选择的集成。
- [接口](https://api.python.langchain.com/en/latest/document_loaders/langchain_core.document_loaders.base.BaseLoader.html):
  基础接口的 API 参考。

## 2. 索引：拆分 {#indexing-split}

我们加载的文档超过42,000个字符。这对于许多模型的上下文窗口来说太长了。即使对于那些能够将完整帖子放入其上下文窗口的模型，模型在处理非常长的输入时也可能会遇到困难。

为了解决这个问题，我们将把`Document`拆分成块以进行嵌入和向量存储。这应该有助于我们在运行时仅检索博客文章中最相关的部分。

在这个例子中，我们将把文档拆分成每块1000个字符，块与块之间重叠200个字符。重叠有助于减轻将陈述与其相关的重要上下文分开的可能性。我们使用
[RecursiveCharacterTextSplitter](/docs/how_to/recursive_text_splitter)，它将使用常见的分隔符（如换行符）递归地拆分文档，直到每块达到适当的大小。这是通用文本用例推荐的文本拆分器。

我们设置`add_start_index=True`，以便在初始文档中每个拆分文档开始的字符索引作为元数据属性“start_index”被保留。

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000, chunk_overlap=200, add_start_index=True
)
all_splits = text_splitter.split_documents(docs)

len(all_splits)
```

```output
66
```

```python
len(all_splits[0].page_content)
```

```output
969
```

```python
all_splits[10].metadata
```

```output
{'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/',
 'start_index': 7056}
```

### 深入了解

`TextSplitter`: 将 `Document` 列表拆分为更小块的对象。`DocumentTransformer` 的子类。

- 通过阅读 [如何文档](/docs/how_to#text-splitters) 了解使用不同方法拆分文本的更多信息
- [代码 (py 或 js)](/docs/integrations/document_loaders/source_code)
- [科学论文](/docs/integrations/document_loaders/grobid)
- [接口](https://api.python.langchain.com/en/latest/base/langchain_text_splitters.base.TextSplitter.html): 基础接口的 API 参考。

`DocumentTransformer`: 对 `Document` 对象列表执行转换的对象。

- [文档](/docs/how_to#text-splitters): 有关如何使用 `DocumentTransformers` 的详细文档
- [集成](/docs/integrations/document_transformers/)
- [接口](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.transformers.BaseDocumentTransformer.html): 基础接口的 API 参考。

## 3. 索引：存储 {#indexing-store}

现在我们需要对我们的 66 个文本块进行索引，以便在运行时可以对它们进行搜索。最常见的方法是嵌入每个文档切分的内容，并将这些嵌入插入到向量数据库（或向量存储）中。当我们想要搜索我们的切分时，我们会获取一个文本搜索查询，对其进行嵌入，并执行某种“相似性”搜索，以识别与我们的查询嵌入最相似的存储切分。最简单的相似性度量是余弦相似性——我们测量每对嵌入（高维向量）之间角度的余弦值。

我们可以使用 [Chroma](/docs/integrations/vectorstores/chroma) 向量存储和 [OpenAIEmbeddings](/docs/integrations/text_embedding/openai) 模型，通过一个命令嵌入并存储所有文档切分。



```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings

vectorstore = Chroma.from_documents(documents=all_splits, embedding=OpenAIEmbeddings())
```

### 深入了解

`Embeddings`: 文本嵌入模型的包装器，用于将文本转换为嵌入。

- [Docs](/docs/how_to/embed_text): 有关如何使用嵌入的详细文档。
- [Integrations](/docs/integrations/text_embedding/): 30多个可供选择的集成。
- [Interface](https://api.python.langchain.com/en/latest/embeddings/langchain_core.embeddings.Embeddings.html): 基础接口的API参考。

`VectorStore`: 向量数据库的包装器，用于存储和查询嵌入。

- [Docs](/docs/how_to/vectorstores): 有关如何使用向量存储的详细文档。
- [Integrations](/docs/integrations/vectorstores/): 40多个可供选择的集成。
- [Interface](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html): 基础接口的API参考。

这完成了管道的**索引**部分。在这一点上，我们拥有一个可查询的向量存储，其中包含我们博客文章的分块内容。给定用户问题，我们理想情况下应该能够返回回答该问题的博客文章片段。

## 4. 检索与生成: Retrieve {#retrieval-and-generation-retrieve}

现在让我们编写实际的应用逻辑。我们想要创建一个简单的应用程序，它接收用户问题，搜索与该问题相关的文档，将检索到的文档和初始问题传递给模型，并返回答案。

首先，我们需要定义检索文档的逻辑。LangChain 定义了一个
[Retriever](/docs/concepts#retrievers/) 接口
，它封装了一个可以根据字符串查询返回相关 `Documents` 的索引。

最常见的 `Retriever` 类型是
[VectorStoreRetriever](/docs/how_to/vectorstore_retriever)，
它利用向量存储的相似性搜索能力来促进检索。任何 `VectorStore` 都可以轻松地通过 `VectorStore.as_retriever()` 转换为 `Retriever`：


```python
retriever = vectorstore.as_retriever(search_type="similarity", search_kwargs={"k": 6})

retrieved_docs = retriever.invoke("What are the approaches to Task Decomposition?")

len(retrieved_docs)
```



```output
6
```



```python
print(retrieved_docs[0].page_content)
```
```output
Tree of Thoughts (Yao et al. 2023) extends CoT by exploring multiple reasoning possibilities at each step. It first decomposes the problem into multiple thought steps and generates multiple thoughts per step, creating a tree structure. The search process can be BFS (breadth-first search) or DFS (depth-first search) with each state evaluated by a classifier (via a prompt) or majority vote.
Task decomposition can be done (1) by LLM with simple prompting like "Steps for XYZ.\n1.", "What are the subgoals for achieving XYZ?", (2) by using task-specific instructions; e.g. "Write a story outline." for writing a novel, or (3) with human inputs.
```

### 深入了解

向量存储通常用于检索，但还有其他检索方式。

`Retriever`: 一个根据文本查询返回 `Document` 的对象

- [文档](/docs/how_to#retrievers): 关于接口和内置检索技术的进一步文档。
  其中包括：
  - `MultiQueryRetriever` [生成输入问题的变体](/docs/how_to/MultiQueryRetriever)，以提高检索命中率。
  - `MultiVectorRetriever` 则生成 [嵌入的变体](/docs/how_to/multi_vector)，同样是为了提高检索命中率。
  - `Max marginal relevance` 在检索到的文档中选择 [相关性和多样性](https://www.cs.cmu.edu/~jgc/publication/The_Use_MMR_Diversity_Based_LTMIR_1998.pdf)，以避免传递重复的上下文。
  - 在向量存储检索期间，可以使用元数据过滤器过滤文档，例如使用 [Self Query Retriever](/docs/how_to/self_query)。
- [集成](/docs/integrations/retrievers/): 与检索服务的集成。
- [接口](https://api.python.langchain.com/en/latest/retrievers/langchain_core.retrievers.BaseRetriever.html): 基础接口的 API 参考。

## 5. 检索与生成：生成 {#retrieval-and-generation-generate}

让我们将所有内容整合成一个链条，接受一个问题，检索相关文档，构建提示，将其传递给模型，并解析输出。

我们将使用 gpt-3.5-turbo OpenAI 聊天模型，但任何 LangChain 的 `LLM` 或 `ChatModel` 都可以替代。

<ChatModelTabs
  customVarName="llm"
  anthropicParams={`"model="claude-3-sonnet-20240229", temperature=0.2, max_tokens=1024"`}
/>

我们将使用一个 RAG 的提示，该提示已检查到 LangChain 提示中心
([这里](https://smith.langchain.com/hub/rlm/rag-prompt))。


```python
from langchain import hub

prompt = hub.pull("rlm/rag-prompt")

example_messages = prompt.invoke(
    {"context": "filler context", "question": "filler question"}
).to_messages()

example_messages
```



```output
[HumanMessage(content="You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.\nQuestion: filler question \nContext: filler context \nAnswer:")]
```



```python
print(example_messages[0].content)
```
```output
You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.
Question: filler question 
Context: filler context 
Answer:
```
我们将使用 [LCEL Runnable](/docs/concepts#langchain-expression-language-lcel)
协议来定义链条，使我们能够 

- 以透明的方式将组件和函数连接在一起 
- 在 LangSmith 中自动追踪我们的链条 
- 直接获得流式、异步和批量调用。

以下是实现代码：


```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough


def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)


rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

for chunk in rag_chain.stream("What is Task Decomposition?"):
    print(chunk, end="", flush=True)
```
```output
Task Decomposition is a process where a complex task is broken down into smaller, more manageable steps or parts. This is often done using techniques like "Chain of Thought" or "Tree of Thoughts", which instruct a model to "think step by step" and transform large tasks into multiple simple tasks. Task decomposition can be prompted in a model, guided by task-specific instructions, or influenced by human inputs.
```
让我们分析一下 LCEL，以理解发生了什么。

首先：这些组件（`retriever`、`prompt`、`llm` 等）都是 [Runnable](/docs/concepts#langchain-expression-language-lcel) 的实例。这意味着它们实现了相同的方法——例如同步和异步的 `.invoke`、`.stream` 或 `.batch`——这使得它们更容易连接在一起。它们可以通过 `|` 操作符连接成一个 [RunnableSequence](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableSequence.html)——另一个 Runnable。

LangChain 会在遇到 `|` 操作符时自动将某些对象转换为 runnable。在这里，`format_docs` 被转换为 [RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html)，而包含 `"context"` 和 `"question"` 的字典被转换为 [RunnableParallel](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableParallel.html)。细节不如更大的观点重要，即每个对象都是一个 Runnable。

让我们追踪输入问题如何通过上述 runnable 流动。

正如我们上面所看到的，`prompt` 的输入预计是一个包含键 `"context"` 和 `"question"` 的字典。因此，这个链条的第一个元素构建了将从输入问题计算出这两个值的 runnables：
- `retriever | format_docs` 将问题传递给检索器，生成 [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html) 对象，然后传递给 `format_docs` 以生成字符串；
- `RunnablePassthrough()` 将输入问题原样传递。

也就是说，如果你构建了
```python
chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
)
```
那么 `chain.invoke(question)` 将构建一个格式化的提示，准备进行推理。（注意：在使用 LCEL 开发时，测试这样的子链可能是实用的。）

链条的最后步骤是 `llm`，它执行推理，以及 `StrOutputParser()`，它只是从 LLM 的输出消息中提取字符串内容。

你可以通过其 [LangSmith 追踪](https://smith.langchain.com/public/1799e8db-8a6d-4eb2-84d5-46e8d7d5a99b/r) 分析这个链条的各个步骤。

### 内置链

如果需要，LangChain 包含实现上述 LCEL 的便捷函数。我们组合了两个函数：

- [create_stuff_documents_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.combine_documents.stuff.create_stuff_documents_chain.html) 指定如何将检索到的上下文输入到提示和 LLM 中。在这种情况下，我们将“填充”内容到提示中——即，我们将包含所有检索到的上下文，而不进行任何总结或其他处理。它在很大程度上实现了我们上面的 `rag_chain`，输入键为 `context` 和 `input`——它使用检索到的上下文和查询生成答案。
- [create_retrieval_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.retrieval.create_retrieval_chain.html) 添加了检索步骤，并将检索到的上下文通过链传播，提供最终答案的同时也包含上下文。它的输入键为 `input`，并在输出中包含 `input`、`context` 和 `answer`。

```python
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_core.prompts import ChatPromptTemplate

system_prompt = (
    "You are an assistant for question-answering tasks. "
    "Use the following pieces of retrieved context to answer "
    "the question. If you don't know the answer, say that you "
    "don't know. Use three sentences maximum and keep the "
    "answer concise."
    "\n\n"
    "{context}"
)

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human", "{input}"),
    ]
)


question_answer_chain = create_stuff_documents_chain(llm, prompt)
rag_chain = create_retrieval_chain(retriever, question_answer_chain)

response = rag_chain.invoke({"input": "What is Task Decomposition?"})
print(response["answer"])
```
```output
Task Decomposition is a process in which complex tasks are broken down into smaller and simpler steps. Techniques like Chain of Thought (CoT) and Tree of Thoughts are used to enhance model performance on these tasks. The CoT method instructs the model to think step by step, decomposing hard tasks into manageable ones, while Tree of Thoughts extends CoT by exploring multiple reasoning possibilities at each step, creating a tree structure of thoughts.
```
#### 返回来源
在问答应用中，通常重要的是向用户展示用于生成答案的来源。LangChain 的内置 `create_retrieval_chain` 将通过 `"context"` 键将检索到的源文档传播到输出中：

```python
for document in response["context"]:
    print(document)
    print()
```

### 深入了解

#### 选择模型

`ChatModel`：一个基于LLM的聊天模型。接收一系列消息并返回一条消息。

- [文档](/docs/how_to#chat-models)
- [集成](/docs/integrations/chat/)：可选择25+个集成。
- [接口](https://api.python.langchain.com/en/latest/language_models/langchain_core.language_models.chat_models.BaseChatModel.html)：基础接口的API参考。

`LLM`：一个文本输入文本输出的LLM。接收一个字符串并返回一个字符串。

- [文档](/docs/how_to#llms)
- [集成](/docs/integrations/llms)：可选择75+个集成。
- [接口](https://api.python.langchain.com/en/latest/language_models/langchain_core.language_models.llms.BaseLLM.html)：基础接口的API参考。

查看关于本地运行模型的RAG指南
[这里](/docs/tutorials/local_rag)。

#### 自定义提示

如上所示，我们可以从提示中心加载提示（例如，[这个RAG提示](https://smith.langchain.com/hub/rlm/rag-prompt)）。提示也可以很容易地自定义：

```python
from langchain_core.prompts import PromptTemplate

template = """Use the following pieces of context to answer the question at the end.
If you don't know the answer, just say that you don't know, don't try to make up an answer.
Use three sentences maximum and keep the answer as concise as possible.
Always say "thanks for asking!" at the end of the answer.

{context}

Question: {question}

Helpful Answer:"""
custom_rag_prompt = PromptTemplate.from_template(template)

rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | custom_rag_prompt
    | llm
    | StrOutputParser()
)

rag_chain.invoke("What is Task Decomposition?")
```

```output
'Task decomposition is the process of breaking down a complex task into smaller, more manageable parts. Techniques like Chain of Thought (CoT) and Tree of Thoughts allow an agent to "think step by step" and explore multiple reasoning possibilities, respectively. This process can be executed by a Language Model with simple prompts, task-specific instructions, or human inputs. Thanks for asking!'
```

查看[LangSmith追踪](https://smith.langchain.com/public/da23c4d8-3b33-47fd-84df-a3a582eedf84/r)

## 下一步

我们已经涵盖了构建基本问答应用程序的步骤：

- 使用 [Document Loader](/docs/concepts#document-loaders) 加载数据
- 使用 [Text Splitter](/docs/concepts#text-splitters) 对索引数据进行切块，使其更易于模型使用
- [Embedding the data](/docs/concepts#embedding-models) 并将数据存储在 [vectorstore](/docs/how_to/vectorstores) 中
- [Retrieving](/docs/concepts#retrievers) 之前存储的块以响应传入的问题
- 使用检索到的块作为上下文生成答案

在上述每个部分中还有许多功能、集成和扩展可供探索。除了上述提到的 **深入了解** 资源，好的下一步包括：

- [Return sources](/docs/how_to/qa_sources): 学习如何返回源文档
- [Streaming](/docs/how_to/streaming): 学习如何流式输出和中间步骤
- [Add chat history](/docs/how_to/message_history): 学习如何将聊天历史添加到您的应用程序中
- [Retrieval conceptual guide](/docs/concepts/#retrieval): 特定检索技术的高级概述
- [Build a local RAG application](/docs/tutorials/local_rag): 创建一个类似于上述应用程序的应用，使用所有本地组件