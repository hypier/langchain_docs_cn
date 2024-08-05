---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/tutorials/pdf_qa.ipynb
keywords: [pdf, 文档加载器]
---

# 构建 PDF 导入和问答系统

:::info 前提条件

本指南假设您熟悉以下概念：

- [文档加载器](/docs/concepts/#document-loaders)
- [聊天模型](/docs/concepts/#chat-models)
- [嵌入](/docs/concepts/#embedding-models)
- [向量存储](/docs/concepts/#vector-stores)
- [检索增强生成](/docs/tutorials/rag/)

:::

PDF 文件通常包含其他来源无法获得的重要非结构化数据。它们可能相当冗长，并且与纯文本文件不同，通常不能直接输入到语言模型的提示中。

在本教程中，您将创建一个可以回答有关 PDF 文件的问题的系统。更具体地说，您将使用 [文档加载器](/docs/concepts/#document-loaders) 加载可供 LLM 使用的文本格式，然后构建一个检索增强生成（RAG）管道来回答问题，包括引用源材料。

本教程将略过一些在我们的 [RAG](/docs/tutorials/rag/) 教程中更深入讨论的概念，因此如果您还没有阅读过这些内容，您可能想先了解一下。

让我们开始吧！

## 加载文档

首先，您需要选择一个要加载的 PDF。我们将使用来自 [Nike 的年度公共 SEC 报告](https://s1.q4cdn.com/806093406/files/doc_downloads/2023/414759-1-_5_Nike-NPS-Combo_Form-10-K_WR.pdf) 的文档。它超过 100 页，包含一些关键数据和较长的解释性文本。不过，您可以自由选择任何 PDF。

一旦您选择了 PDF，下一步是将其加载到 LLM 更容易处理的格式中，因为 LLM 通常需要文本输入。LangChain 有一些不同的 [内置文档加载器](/docs/how_to/document_loader_pdf/) 可供您实验。下面，我们将使用一个由 [`pypdf`](https://pypi.org/project/pypdf/) 包驱动的加载器，它从文件路径读取：

```python
%pip install -qU pypdf langchain_community
```

```python
from langchain_community.document_loaders import PyPDFLoader

file_path = "../example_data/nke-10k-2023.pdf"
loader = PyPDFLoader(file_path)

docs = loader.load()

print(len(docs))
```
```output
107
```

```python
print(docs[0].page_content[0:100])
print(docs[0].metadata)
```
```output
Table of Contents
UNITED STATES
SECURITIES AND EXCHANGE COMMISSION
Washington, D.C. 20549
FORM 10-K

{'source': '../example_data/nke-10k-2023.pdf', 'page': 0}
```
那么刚刚发生了什么？

- 加载器从指定路径读取 PDF 到内存中。
- 然后，它使用 `pypdf` 包提取文本数据。
- 最后，它为 PDF 的每一页创建一个 LangChain [文档](/docs/concepts/#documents)，其中包含页面内容和一些关于文本来源的元数据。

LangChain 还有 [许多其他文档加载器](/docs/integrations/document_loaders/) 适用于其他数据源，或者您可以创建一个 [自定义文档加载器](/docs/how_to/document_loader_custom/)。

## 使用 RAG 的问答

接下来，您将为后续检索准备加载的文档。使用 [text splitter](/docs/concepts/#text-splitters)，您将把加载的文档拆分为更小的文档，以便更容易适应 LLM 的上下文窗口，然后将它们加载到 [vector store](/docs/concepts/#vector-stores) 中。然后，您可以从向量存储中创建一个 [retriever](/docs/concepts/#retrievers)，以便在我们的 RAG 链中使用：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" openaiParams={`model="gpt-4o"`} />


```python
%pip install langchain_chroma langchain_openai
```


```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
splits = text_splitter.split_documents(docs)
vectorstore = Chroma.from_documents(documents=splits, embedding=OpenAIEmbeddings())

retriever = vectorstore.as_retriever()
```

最后，您将使用一些内置助手来构建最终的 `rag_chain`：


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

results = rag_chain.invoke({"input": "What was Nike's revenue in 2023?"})

results
```





您可以看到，结果字典的 `answer` 键中得到了最终答案，以及 LLM 用于生成答案的 `context`。

进一步检查 `context` 下的值，您可以看到它们是每个包含摄取页面内容块的文档。值得注意的是，这些文档还保留了您首次加载时的原始元数据：


```python
print(results["context"][0].page_content)
```
```output
目录
2023 财年耐克品牌收入亮点
以下表格按可报告的运营部门、分销渠道和主要产品线列出了耐克品牌的收入：
2023 财年与 2022 财年比较
•耐克公司2023 财年的收入为 512 亿美元，较 2022 财年分别增长了 10% 和 16%（按报告和货币中性计算）。
增长主要得益于北美、欧洲、中东和非洲（“EMEA”）、APLA 和大中华区的收入增长，分别对耐克公司的收入贡献了约 7%、6%、2% 和 1 个百分点。
•耐克品牌的收入占耐克公司收入的90%以上，按报告和货币中性计算，分别增长了10%和16%。这一增长主要得益于男装、乔丹品牌、女装和儿童装的收入增长，分别在批发等效基础上增长了17%、35%、11%和10%。
```

```python
print(results["context"][0].metadata)
```
```output
{'page': 35, 'source': '../example_data/nke-10k-2023.pdf'}
```
这一特定块来自原始 PDF 的第 35 页。您可以使用这些数据来显示答案来自 PDF 的哪一页，从而使用户能够快速验证答案是否基于源材料。

:::info
要深入了解 RAG，请查看 [这个更专注的教程](/docs/tutorials/rag/) 或 [我们的操作指南](/docs/how_to/#qa-with-rag)。
:::

## 下一步

您现在已经了解了如何使用文档加载器从 PDF 文件中加载文档，以及一些可以用来准备加载数据以进行 RAG 的技术。

有关文档加载器的更多信息，您可以查看：

- [概念指南中的条目](/docs/concepts/#document-loaders)
- [相关的操作指南](/docs/how_to/#document-loaders)
- [可用的集成](/docs/integrations/document_loaders/)
- [如何创建自定义文档加载器](/docs/how_to/document_loader_custom/)

有关 RAG 的更多信息，请参见：

- [构建检索增强生成 (RAG) 应用程序](/docs/tutorials/rag/)
- [相关的操作指南](/docs/how_to/#qa-with-rag)