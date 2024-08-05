---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/tutorials/local_rag.ipynb
---

# 构建本地 RAG 应用程序

:::info 先决条件

本指南假设您熟悉以下概念：

- [聊天模型](/docs/concepts/#chat-models)
- [链式可运行项](/docs/how_to/sequence/)
- [嵌入](/docs/concepts/#embedding-models)
- [向量存储](/docs/concepts/#vector-stores)
- [检索增强生成](/docs/tutorials/rag/)

:::

像 [llama.cpp](https://github.com/ggerganov/llama.cpp)、[Ollama](https://github.com/ollama/ollama) 和 [llamafile](https://github.com/Mozilla-Ocho/llamafile) 这样的项目的流行突显了在本地运行 LLM 的重要性。

LangChain 与 [许多开源 LLM 提供商](/docs/how_to/local_llms) 集成，可以在本地运行。

本指南将展示如何通过一个提供商 [Ollama](/docs/integrations/providers/ollama/) 在本地（例如，在您的笔记本电脑上）使用本地嵌入和本地 LLM 运行 `LLaMA 3.1`。但是，如果您愿意，可以设置并切换到其他本地提供商，例如 [LlamaCPP](/docs/integrations/chat/llamacpp/)。

**注意：** 本指南使用一个 [聊天模型](/docs/concepts/#chat-models) 包装器，它负责为您使用的特定本地模型格式化输入提示。然而，如果您直接使用 [文本输入/文本输出 LLM](/docs/concepts/#llms) 包装器提示本地模型，则可能需要使用针对您特定模型的提示。这通常 [需要包含特殊标记](https://huggingface.co/blog/llama2#how-to-prompt-llama-2)。 [这是 LLaMA 2 的一个示例](https://smith.langchain.com/hub/rlm/rag-prompt-llama)。

## 设置

首先，我们需要设置 Ollama。

[他们的 GitHub 仓库](https://github.com/ollama/ollama) 提供了详细的说明，我们在这里总结如下：

- [下载](https://ollama.com/download) 并运行他们的桌面应用程序
- 从命令行中，从 [这个选项列表](https://ollama.com/library) 获取模型。对于本指南，您需要：
  - 一个通用模型，如 `llama3.1:8b`，可以通过类似 `ollama pull llama3.1:8b` 的命令获取
  - 一个 [文本嵌入模型](https://ollama.com/search?c=embedding)，如 `nomic-embed-text`，可以通过类似 `ollama pull nomic-embed-text` 的命令获取
- 当应用程序运行时，所有模型会自动在 `localhost:11434` 上提供
- 请注意，您的模型选择将取决于您的硬件能力

接下来，安装本地嵌入、向量存储和推理所需的包。

```python
# Document loading, retrieval methods and text splitting
%pip install -qU langchain langchain_community

# Local vector store via Chroma
%pip install -qU langchain_chroma

# Local inference and embeddings via Ollama
%pip install -qU langchain_ollama
```

您还可以 [查看此页面](/docs/integrations/text_embedding/) 以获取可用嵌入模型的完整列表

## 文档加载

现在让我们加载并拆分一个示例文档。

我们将使用Lilian Weng关于代理的[博客文章](https://lilianweng.github.io/posts/2023-06-23-agent/)作为示例。

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import WebBaseLoader

loader = WebBaseLoader("https://lilianweng.github.io/posts/2023-06-23-agent/")
data = loader.load()

text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
all_splits = text_splitter.split_documents(data)
```

接下来，以下步骤将初始化您的向量存储。我们使用[`nomic-embed-text`](https://ollama.com/library/nomic-embed-text)，但您也可以探索其他提供者或选项：

```python
from langchain_chroma import Chroma
from langchain_ollama import OllamaEmbeddings

local_embeddings = OllamaEmbeddings(model="nomic-embed-text")

vectorstore = Chroma.from_documents(documents=all_splits, embedding=local_embeddings)
```

现在我们有一个可用的向量存储！测试相似性搜索是否正常工作：

```python
question = "任务分解的方法有哪些？"
docs = vectorstore.similarity_search(question)
len(docs)
```

```output
4
```

```python
docs[0]
```

```output
Document(metadata={'description': '使用LLM（大型语言模型）作为核心控制器构建代理是一个很酷的概念。一些概念验证演示，如AutoGPT、GPT-Engineer和BabyAGI，作为激励的例子。LLM的潜力不仅限于生成写得好的副本、故事、论文和程序；它可以被视为一个强大的通用问题解决者。\n代理系统概述 在一个由LLM驱动的自主代理系统中，LLM充当代理的大脑，辅以几个关键组件：', 'language': 'en', 'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/', 'title': "LLM Powered Autonomous Agents | Lil'Log"}, page_content='任务分解可以通过（1）LLM简单提示如“XYZ的步骤。\\n1.”、“实现XYZ的子目标是什么？”来完成；（2）使用特定任务的指令；例如，“为写小说写一个故事大纲。”或（3）通过人类输入。')
```

接下来，设置一个模型。我们在这里使用Ollama的`llama3.1:8b`，但您可以[探索其他提供者](/docs/how_to/local_llms/)或根据您的硬件设置[选择模型选项](https://ollama.com/library)：

```python
from langchain_ollama import ChatOllama

model = ChatOllama(
    model="llama3.1:8b",
)
```

测试一下，以确保您已正确设置一切：

```python
response_message = model.invoke(
    "模拟Stephen Colbert和John Oliver之间的说唱对决"
)

print(response_message.content)
```
```output
**场景设定：一个座无虚席的竞技场，观众热情高涨。在蓝色角落，我们有Stephen Colbert，也就是“奥赖利因素”本人。在红色角落，挑战者John Oliver。评委是Tina Fey、Larry Wilmore和Patton Oswalt。观众欢呼，双方对峙。**

**Stephen Colbert（又名“真相与扭曲”）：**
Yo，我是讽刺的国王，人人都怕我
我的节目在深夜，但我的笑话很清晰
我用精确和力量揭穿政客
他们在我机智的面前，白天黑夜都在颤抖

**John Oliver：**
等等，斯蒂夫，你可能曾经风光无限
但我是新来的小子，拥有不同的巅峰
是时候从那90年代的昏迷中醒来了，儿子
我的节目有锐度，我的事实永远不会结束

**Stephen Colbert：**
哦，所以你认为你是那个“上周”之冠
但你的笑话过时，像我曾经用过的那样
我是荒谬的主人，旋转的领主
你不过是个英国进口，试图融入

**John Oliver：**
斯蒂夫，我的朋友，你也许是第一个
但我有技巧和机智，从未模糊
我的节目不怕迎接挑战
我将让你思考，无论发生什么

**Stephen Colbert：**
好吧，时候到了，就像两个老朋友
让我们看看谁的讽刺至高无上，直到最后
但我有一个秘密，可能会决定你的命运
我的幽默具有传染性，已经为时已晚！

**John Oliver：**
来吧，斯蒂夫！我准备好迎接你
我将应对你的笑话，向他们展示该怎么做
我的讽刺锋利，如夜晚的手术刀
你只是过去的遗物，没有一场战斗

**评委们在考虑，权衡韵律和节奏。最后，他们宣布了他们的决定：**

Tina Fey：我得选John Oliver。他的笑话更尖锐，表达更流畅。

Larry Wilmore：同意！但Stephen Colbert仍然保留那种老派魅力。

Patton Oswalt：你知道吗？这是一场平局。他们都带来了热度！

**观众疯狂欢呼，双方鞠躬。说唱对决可能结束了，但讽刺的战争才刚刚开始...**
```

## 在链中使用

我们可以通过传入检索到的文档和一个简单的提示，使用任一模型创建一个摘要链。

它使用提供的输入键值格式化提示模板，并将格式化后的字符串传递给指定的模型：

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_template(
    "Summarize the main themes in these retrieved docs: {docs}"
)


# Convert loaded documents into strings by concatenating their content
# and ignoring metadata
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)


chain = {"docs": format_docs} | prompt | model | StrOutputParser()

question = "What are the approaches to Task Decomposition?"

docs = vectorstore.similarity_search(question)

chain.invoke(docs)
```


```output
'The main themes in these documents are:\n\n1. **Task Decomposition**: The process of breaking down complex tasks into smaller, manageable subgoals is crucial for efficient task handling.\n2. **Autonomous Agent System**: A system powered by Large Language Models (LLMs) that can perform planning, reflection, and refinement to improve the quality of final results.\n3. **Challenges in Planning and Decomposition**:\n\t* Long-term planning and task decomposition are challenging for LLMs.\n\t* Adjusting plans when faced with unexpected errors is difficult for LLMs.\n\t* Humans learn from trial and error, making them more robust than LLMs in certain situations.\n\nOverall, the documents highlight the importance of task decomposition and planning in autonomous agent systems powered by LLMs, as well as the challenges that still need to be addressed.'
```

## 问答

您还可以使用本地模型和向量存储进行问答。以下是一个简单字符串提示的示例：

```python
from langchain_core.runnables import RunnablePassthrough

RAG_TEMPLATE = """
You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.

<context>
{context}
</context>

Answer the following question:

{question}"""

rag_prompt = ChatPromptTemplate.from_template(RAG_TEMPLATE)

chain = (
    RunnablePassthrough.assign(context=lambda input: format_docs(input["context"]))
    | rag_prompt
    | model
    | StrOutputParser()
)

question = "What are the approaches to Task Decomposition?"

docs = vectorstore.similarity_search(question)

# Run
chain.invoke({"context": docs, "question": question})
```

```output
'任务分解可以通过 (1) 使用 LLM 的简单提示，(2) 任务特定的指令，或 (3) 人工输入来完成。这种方法有助于将大型任务分解为更小、可管理的子目标，以便高效处理复杂任务。它使代理能够提前规划，并通过反思和改进提高最终结果的质量。'
```

## Q&A 与检索

最后，您可以根据用户问题自动从我们的向量存储中检索文档，而不是手动传入文档：

```python
retriever = vectorstore.as_retriever()

qa_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | rag_prompt
    | model
    | StrOutputParser()
)
```


```python
question = "Task Decomposition 的方法有哪些？"

qa_chain.invoke(question)
```



```output
'Task decomposition can be done through (1) simple prompting in Large Language Models (LLM), (2) using task-specific instructions, or (3) with human inputs. This process involves breaking down large tasks into smaller, manageable subgoals for efficient handling of complex tasks.'
```

## 下一步

您现在已经了解了如何使用所有本地组件构建 RAG 应用程序。RAG 是一个非常深奥的话题，您可能会对以下指南感兴趣，这些指南讨论和演示了其他技术：

- [视频：使用 LLaMA 3 构建可靠的完全本地 RAG 代理](https://www.youtube.com/watch?v=-ROS6gfYIts)，适用于本地模型的代理方法
- [视频：从零开始使用开源本地 LLM 构建纠正性 RAG](https://www.youtube.com/watch?v=E2shqsYwxck)
- [检索的概念指南](/docs/concepts/#retrieval)，概述了您可以应用以提高性能的各种检索技术
- [RAG 的操作指南](/docs/how_to/#qa-with-rag)，深入探讨 RAG 的不同细节
- [如何在本地运行模型](/docs/how_to/local_llms/)，提供设置不同提供者的不同方法