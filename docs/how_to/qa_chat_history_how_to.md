---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/qa_chat_history_how_to.ipynb
---

# 如何添加聊天记录

在许多问答应用程序中，我们希望允许用户进行来回对话，这意味着应用程序需要某种形式的“记忆”，以存储过去的问题和答案，并需要一些逻辑将这些内容融入当前的思考中。

在本指南中，我们重点关注**添加逻辑以整合历史消息。**

这在很大程度上是[对话式 RAG 教程](/docs/tutorials/qa_chat_history)的简化版本。

我们将涵盖两种方法：
1. [链式方法](/docs/how_to/qa_chat_history_how_to#chains)，在其中我们始终执行检索步骤；
2. [代理方法](/docs/how_to/qa_chat_history_how_to#agents)，在其中我们赋予 LLM 自主决定是否以及如何执行检索步骤（或多个步骤）。

对于外部知识源，我们将使用 Lilian Weng 的[LLM 驱动的自主代理](https://lilianweng.github.io/posts/2023-06-23-agent/)博客文章，该文章来自[RAG 教程](/docs/tutorials/rag)。

## 设置

### 依赖

在本教程中，我们将使用 OpenAI embeddings 和 Chroma 向量存储，但此处展示的所有内容适用于任何 [Embeddings](/docs/concepts#embedding-models)、[VectorStore](/docs/concepts#vectorstores) 或 [Retriever](/docs/concepts#retrievers)。

我们将使用以下软件包：

```python
%%capture --no-stderr
%pip install --upgrade --quiet  langchain langchain-community langchain-chroma bs4
```

我们需要设置环境变量 `OPENAI_API_KEY`，可以直接设置或从 `.env` 文件中加载，如下所示：

```python
import getpass
import os

if not os.environ.get("OPENAI_API_KEY"):
    os.environ["OPENAI_API_KEY"] = getpass.getpass()

# import dotenv

# dotenv.load_dotenv()
```

### LangSmith

您使用 LangChain 构建的许多应用程序将包含多个步骤和多次调用 LLM。随着这些应用程序变得越来越复杂，能够检查链或代理内部究竟发生了什么变得至关重要。做到这一点的最佳方法是使用 [LangSmith](https://smith.langchain.com)。

请注意，LangSmith 不是必需的，但它是有帮助的。如果您确实想使用 LangSmith，在您在上述链接注册后，请确保设置您的环境变量以开始记录跟踪：


```python
os.environ["LANGCHAIN_TRACING_V2"] = "true"
if not os.environ.get("LANGCHAIN_API_KEY"):
    os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## Chains {#chains}

在对话式RAG应用中，发给检索器的查询应受对话上下文的影响。LangChain提供了一个[create_history_aware_retriever](https://api.python.langchain.com/en/latest/chains/langchain.chains.history_aware_retriever.create_history_aware_retriever.html)构造函数来简化这一过程。它构建了一个链，接受`input`和`chat_history`作为输入，并具有与检索器相同的输出模式。`create_history_aware_retriever`需要以下输入：  

1. LLM；
2. Retriever；
3. Prompt。

首先，我们获取这些对象：

### LLM

我们可以使用任何支持的聊天模型：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />

### 检索器

对于检索器，我们将使用 [WebBaseLoader](https://api.python.langchain.com/en/latest/document_loaders/langchain_community.document_loaders.web_base.WebBaseLoader.html) 来加载网页内容。在这里，我们实例化一个 `Chroma` 向量存储，然后使用其 [.as_retriever](https://api.python.langchain.com/en/latest/vectorstores/langchain_core.vectorstores.VectorStore.html#langchain_core.vectorstores.VectorStore.as_retriever) 方法构建一个可以纳入 [LCEL](/docs/concepts/#langchain-expression-language) 链的检索器。

```python
import bs4
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

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
retriever = vectorstore.as_retriever()
```

### 提示

我们将使用一个包含 `MessagesPlaceholder` 变量的提示，名为 "chat_history"。这允许我们通过 "chat_history" 输入键将消息列表传递给提示，这些消息将在系统消息之后和包含最新问题的人类消息之前插入。

```python
from langchain.chains import create_history_aware_retriever
from langchain_core.prompts import MessagesPlaceholder

contextualize_q_system_prompt = (
    "Given a chat history and the latest user question "
    "which might reference context in the chat history, "
    "formulate a standalone question which can be understood "
    "without the chat history. Do NOT answer the question, "
    "just reformulate it if needed and otherwise return it as is."
)

contextualize_q_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", contextualize_q_system_prompt),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
    ]
)
```

### 组装链条

我们可以实例化带历史感知的检索器：

```python
history_aware_retriever = create_history_aware_retriever(
    llm, retriever, contextualize_q_prompt
)
```

该链条将输入查询的重新表述添加到我们的检索器之前，以便检索时能够结合对话的上下文。

现在我们可以构建完整的问答链。

如同[RAG教程](/docs/tutorials/rag)中所示，我们将使用[create_stuff_documents_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.combine_documents.stuff.create_stuff_documents_chain.html)来生成一个`question_answer_chain`，其输入键为`context`、`chat_history`和`input`——它接受检索到的上下文以及对话历史和查询，以生成答案。

我们用[create_retrieval_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.retrieval.create_retrieval_chain.html)构建最终的`rag_chain`。该链条按顺序应用`history_aware_retriever`和`question_answer_chain`，保留中间输出，例如检索到的上下文，以便于使用。它的输入键为`input`和`chat_history`，输出中包含`input`、`chat_history`、`context`和`answer`。

```python
system_prompt = (
    "You are an assistant for question-answering tasks. "
    "Use the following pieces of retrieved context to answer "
    "the question. If you don't know the answer, say that you "
    "don't know. Use three sentences maximum and keep the "
    "answer concise."
    "\n\n"
    "{context}"
)
qa_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
    ]
)
question_answer_chain = create_stuff_documents_chain(llm, qa_prompt)

rag_chain = create_retrieval_chain(history_aware_retriever, question_answer_chain)
```

### 添加聊天记录

要管理聊天记录，我们需要：

1. 一个用于存储聊天记录的对象；
2. 一个包装我们的链并管理聊天记录更新的对象。

为此，我们将使用 [BaseChatMessageHistory](https://api.python.langchain.com/en/latest/chat_history/langchain_core.chat_history.BaseChatMessageHistory.html) 和 [RunnableWithMessageHistory](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.history.RunnableWithMessageHistory.html)。后者是一个 LCEL 链和 `BaseChatMessageHistory` 的包装器，负责将聊天记录注入输入并在每次调用后更新它。

有关如何将这些类一起使用以创建有状态对话链的详细指南，请参阅 [如何添加消息历史（记忆）](/docs/how_to/message_history/) LCEL 指南。

下面，我们实现了第二种选项的简单示例，其中聊天记录存储在一个简单的字典中。LangChain 管理与 [Redis](/docs/integrations/memory/redis_chat_message_history/) 和其他技术的内存集成，以提供更强大的持久性。

`RunnableWithMessageHistory` 的实例为您管理聊天记录。它们接受一个配置，其中包含一个键（默认值为 `"session_id"`），指定要获取并预先添加到输入的会话历史，并将输出附加到同一会话历史。以下是一个示例：

```python
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_core.chat_history import BaseChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

store = {}


def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]


conversational_rag_chain = RunnableWithMessageHistory(
    rag_chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history",
    output_messages_key="answer",
)
```

```python
conversational_rag_chain.invoke(
    {"input": "What is Task Decomposition?"},
    config={
        "configurable": {"session_id": "abc123"}
    },  # constructs a key "abc123" in `store`.
)["answer"]
```

```output
'任务分解涉及将复杂任务分解为更小和更简单的步骤，以使其更易于管理和完成。此过程可以使用诸如思维链（CoT）或思维树等技术来指导模型有效地分解任务。通过向语言模型提供简单提示、特定任务的指令或人类输入，可以促进任务分解。'
```

```python
conversational_rag_chain.invoke(
    {"input": "What are common ways of doing it?"},
    config={"configurable": {"session_id": "abc123"}},
)["answer"]
```

```output
'任务分解可以通过多种方法实现，包括使用诸如思维链（CoT）或思维树等技术来有效地指导模型分解任务。常见的任务分解方法包括向语言模型提供简单提示、特定任务的指令或人类输入，以将复杂任务分解为更小和更易管理的步骤。此外，任务分解还可以利用互联网访问等资源进行信息收集、长期记忆管理以及利用 GPT-3.5 驱动的代理进行简单任务的委派。'
```

可以在 `store` 字典中检查对话历史：

```python
from langchain_core.messages import AIMessage

for message in store["abc123"].messages:
    if isinstance(message, AIMessage):
        prefix = "AI"
    else:
        prefix = "用户"

    print(f"{prefix}: {message.content}\n")
```

```output
用户: What is Task Decomposition?

AI: 任务分解涉及将复杂任务分解为更小和更简单的步骤，以使其更易于管理和完成。此过程可以使用诸如思维链（CoT）或思维树等技术来指导模型有效地分解任务。通过向语言模型提供简单提示、特定任务的指令或人类输入，可以促进任务分解。

用户: What are common ways of doing it?

AI: 任务分解可以通过多种方法实现，包括使用诸如思维链（CoT）或思维树等技术来有效地指导模型分解任务。常见的任务分解方法包括向语言模型提供简单提示、特定任务的指令或人类输入，以将复杂任务分解为更小和更易管理的步骤。此外，任务分解还可以利用互联网访问等资源进行信息收集、长期记忆管理以及利用 GPT-3.5 驱动的代理进行简单任务的委派。
```

### 整合起来

![](../../static/img/conversational_retrieval_chain.png)

为了方便起见，我们将所有必要的步骤整合到一个代码单元中：


```python
import bs4
from langchain.chains import create_history_aware_retriever, create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_chroma import Chroma
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.chat_history import BaseChatMessageHistory
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)


### 构建检索器 ###
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
retriever = vectorstore.as_retriever()


### 上下文化问题 ###
contextualize_q_system_prompt = (
    "给定聊天历史和最新的用户问题，"
    "该问题可能引用聊天历史中的上下文，"
    "请形成一个独立的问题，使其可以被理解，"
    "而无需聊天历史。不要回答这个问题，"
    "如有必要请重新表述，其他情况下请原样返回。"
)
contextualize_q_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", contextualize_q_system_prompt),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
    ]
)
history_aware_retriever = create_history_aware_retriever(
    llm, retriever, contextualize_q_prompt
)


### 回答问题 ###
system_prompt = (
    "你是一个问答任务的助手。"
    "请使用以下检索到的上下文来回答"
    "这个问题。如果你不知道答案，请说你"
    "不知道。请最多使用三句话，并保持"
    "回答简洁。"
    "\n\n"
    "{context}"
)
qa_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
    ]
)
question_answer_chain = create_stuff_documents_chain(llm, qa_prompt)

rag_chain = create_retrieval_chain(history_aware_retriever, question_answer_chain)


### 有状态管理聊天历史 ###
store = {}


def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]


conversational_rag_chain = RunnableWithMessageHistory(
    rag_chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history",
    output_messages_key="answer",
)
```


```python
conversational_rag_chain.invoke(
    {"input": "什么是任务分解？"},
    config={
        "configurable": {"session_id": "abc123"}
    },  # 在 `store` 中构建一个键 "abc123"。
)["answer"]
```



```output
'任务分解涉及将复杂任务拆分为更小、更简单的步骤，以使其更易于管理。像思维链（Chain of Thought, CoT）和思维树（Tree of Thoughts）这样的技术有助于将困难任务分解为多个可管理的任务，指导模型逐步思考并在每一步探索多个推理可能性。任务分解可以通过各种方法实现，例如使用提示技术、特定于任务的指令或人类输入。'
```



```python
conversational_rag_chain.invoke(
    {"input": "常见的做法有哪些？"},
    config={"configurable": {"session_id": "abc123"}},
)["answer"]
```



```output
'任务分解的常见方法包括使用提示技术，如思维链（Chain of Thought, CoT）或思维树（Tree of Thoughts），指导模型逐步思考并在每一步探索多个推理可能性。另一种方法是提供特定于任务的指令，例如要求“写一个故事大纲”以指导小说写作的分解过程。此外，任务分解还可以涉及人类输入，以将复杂任务拆分为更小、更简单的步骤。'
```

## 代理 {#agents}

代理利用 LLM 的推理能力在执行过程中做出决策。使用代理可以让您在检索过程中卸载一些自由裁量权。尽管它们的行为比链条更不可预测，但在这种情况下它们提供了一些优势：
- 代理直接生成检索器的输入，而不一定需要我们像上面那样明确构建上下文；
- 代理可以为查询执行多个检索步骤，或者完全不执行检索步骤（例如，作为对用户的通用问候的回应）。

### 检索工具

代理可以访问“工具”并管理它们的执行。在这种情况下，我们将把我们的检索器转换为一个 LangChain 工具，以便代理使用：

```python
from langchain.tools.retriever import create_retriever_tool

tool = create_retriever_tool(
    retriever,
    "blog_post_retriever",
    "Searches and returns excerpts from the Autonomous Agents blog post.",
)
tools = [tool]
```

### 代理构造函数

现在我们已经定义了工具和 LLM，我们可以创建代理。我们将使用 [LangGraph](/docs/concepts/#langgraph) 来构建代理。 
目前我们使用的是高级接口来构建代理，但 LangGraph 的一个优点是，如果您想修改代理逻辑，这个高级接口是由低级、高度可控的 API 支持的。


```python
from langgraph.prebuilt import create_react_agent

agent_executor = create_react_agent(llm, tools)
```

我们现在可以试一试。请注意，到目前为止它是无状态的（我们仍然需要添加内存）


```python
from langchain_core.messages import HumanMessage

query = "什么是任务分解？"

for s in agent_executor.stream(
    {"messages": [HumanMessage(content=query)]},
):
    print(s)
    print("----")
```
```output
Error in LangChainTracer.on_tool_end callback: TracerException("Found chain run at ID 5cd28d13-88dd-4eac-a465-3770ac27eff6, but expected {'tool'} run.")
``````output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_TbhPPPN05GKi36HLeaN4QM90', 'function': {'arguments': '{"query":"Task Decomposition"}', 'name': 'blog_post_retriever'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 19, 'prompt_tokens': 68, 'total_tokens': 87}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': None, 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-2e60d910-879a-4a2a-b1e9-6a6c5c7d7ebc-0', tool_calls=[{'name': 'blog_post_retriever', 'args': {'query': 'Task Decomposition'}, 'id': 'call_TbhPPPN05GKi36HLeaN4QM90'}])]}}
----
{'tools': {'messages': [ToolMessage(content='图 1. LLM 驱动的自主代理系统概述。\n组件一：规划#\n复杂任务通常涉及许多步骤。代理需要知道它们是什么并提前规划。\n任务分解#\n思维链（CoT；Wei 等，2022）已成为增强模型在复杂任务上表现的标准提示技术。模型被指示“逐步思考”，以利用更多的测试时计算将困难任务分解为更小和更简单的步骤。CoT 将大任务转化为多个可管理的任务，并为模型的思维过程提供了一个解释。\n\n图 1. LLM 驱动的自主代理系统概述。\n组件一：规划#\n复杂任务通常涉及许多步骤。代理需要知道它们是什么并提前规划。\n任务分解#\n思维链（CoT；Wei 等，2022）已成为增强模型在复杂任务上表现的标准提示技术。模型被指示“逐步思考”，以利用更多的测试时计算将困难任务分解为更小和更简单的步骤。CoT 将大任务转化为多个可管理的任务，并为模型的思维过程提供了一个解释。\n\n思维树（Yao 等，2023）通过探索每一步的多种推理可能性扩展了 CoT。它首先将问题分解为多个思维步骤，并为每一步生成多个思维，创建一个树结构。搜索过程可以是 BFS（广度优先搜索）或 DFS（深度优先搜索），每个状态通过分类器（通过提示）或多数投票进行评估。\n任务分解可以通过 (1) LLM 使用简单提示，如 "步骤为 XYZ.\\n1."，"实现 XYZ 的子目标是什么？"，(2) 使用特定任务的指令；例如 "写一个故事大纲。" 用于写小说，或 (3) 通过人类输入来完成。\n\n思维树（Yao 等，2023）通过探索每一步的多种推理可能性扩展了 CoT。它首先将问题分解为多个思维步骤，并为每一步生成多个思维，创建一个树结构。搜索过程可以是 BFS（广度优先搜索）或 DFS（深度优先搜索），每个状态通过分类器（通过提示）或多数投票进行评估。\n任务分解可以通过 (1) LLM 使用简单提示，如 "步骤为 XYZ.\\n1."，"实现 XYZ 的子目标是什么？"，(2) 使用特定任务的指令；例如 "写一个故事大纲。" 用于写小说，或 (3) 通过人类输入来完成。', name='blog_post_retriever', tool_call_id='call_TbhPPPN05GKi36HLeaN4QM90')]}}
----
{'agent': {'messages': [AIMessage(content='任务分解是一种将复杂任务分解为更小和更简单步骤的技术。这种方法有助于将大任务转化为多个可管理的任务，使自主代理更容易处理和理解思维过程。任务分解的一个常见方法是思维链（CoT）技术，模型被指示“逐步思考”以分解困难任务。CoT 的另一个扩展是思维树，它通过创建每一步的多个思维树结构，探索每一步的多种推理可能性。任务分解可以通过多种方法来实现，例如使用简单提示、特定任务的指令或人类输入。', response_metadata={'token_usage': {'completion_tokens': 130, 'prompt_tokens': 636, 'total_tokens': 766}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-3ef17638-65df-4030-a7fe-795e6da91c69-0')]}}
----
```
LangGraph 内置了持久性，因此我们不需要使用 ChatMessageHistory！相反，我们可以直接将检查点传递给我们的 LangGraph 代理。

通过在配置字典中指定对话线程的键来管理不同的对话，如下所示。


```python
from langgraph.checkpoint.sqlite import SqliteSaver

memory = SqliteSaver.from_conn_string(":memory:")

agent_executor = create_react_agent(llm, tools, checkpointer=memory)
```

这就是构建对话 RAG 代理所需的全部内容。

让我们观察它的行为。请注意，如果我们输入一个不需要检索步骤的查询，代理不会执行一个：


```python
config = {"configurable": {"thread_id": "abc123"}}

for s in agent_executor.stream(
    {"messages": [HumanMessage(content="嗨！我是鲍勃")]}, config=config
):
    print(s)
    print("----")
```
```output
{'agent': {'messages': [AIMessage(content='你好，鲍勃！我今天能帮你什么？', response_metadata={'token_usage': {'completion_tokens': 11, 'prompt_tokens': 67, 'total_tokens': 78}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-1cd17562-18aa-4839-b41b-403b17a0fc20-0')]}}
----
```
此外，如果我们输入一个确实需要检索步骤的查询，代理会生成对工具的输入：


```python
query = "什么是任务分解？"

for s in agent_executor.stream(
    {"messages": [HumanMessage(content=query)]}, config=config
):
    print(s)
    print("----")
```
```output
Error in LangChainTracer.on_tool_end callback: TracerException("Found chain run at ID c54381c0-c5d9-495a-91a0-aca4ae755663, but expected {'tool'} run.")
``````output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_rg7zKTE5e0ICxVSslJ1u9LMg', 'function': {'arguments': '{"query":"Task Decomposition"}', 'name': 'blog_post_retriever'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 19, 'prompt_tokens': 91, 'total_tokens': 110}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': None, 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-122bf097-7ff1-49aa-b430-e362b51354ad-0', tool_calls=[{'name': 'blog_post_retriever', 'args': {'query': 'Task Decomposition'}, 'id': 'call_rg7zKTE5e0ICxVSslJ1u9LMg'}])]}}
----
{'tools': {'messages': [ToolMessage(content='图 1. LLM 驱动的自主代理系统概述。\n组件一：规划#\n复杂任务通常涉及许多步骤。代理需要知道它们是什么并提前规划。\n任务分解#\n思维链（CoT；Wei 等，2022）已成为增强模型在复杂任务上表现的标准提示技术。模型被指示“逐步思考”，以利用更多的测试时计算将困难任务分解为更小和更简单的步骤。CoT 将大任务转化为多个可管理的任务，并为模型的思维过程提供了一个解释。\n\n图 1. LLM 驱动的自主代理系统概述。\n组件一：规划#\n复杂任务通常涉及许多步骤。代理需要知道它们是什么并提前规划。\n任务分解#\n思维链（CoT；Wei 等，2022）已成为增强模型在复杂任务上表现的标准提示技术。模型被指示“逐步思考”，以利用更多的测试时计算将困难任务分解为更小和更简单的步骤。CoT 将大任务转化为多个可管理的任务，并为模型的思维过程提供了一个解释。\n\n思维树（Yao 等，2023）通过探索每一步的多种推理可能性扩展了 CoT。它首先将问题分解为多个思维步骤，并为每一步生成多个思维，创建一个树结构。搜索过程可以是 BFS（广度优先搜索）或 DFS（深度优先搜索），每个状态通过分类器（通过提示）或多数投票进行评估。\n任务分解可以通过 (1) LLM 使用简单提示，如 "步骤为 XYZ.\\n1."，"实现 XYZ 的子目标是什么？"，(2) 使用特定任务的指令；例如 "写一个故事大纲。" 用于写小说，或 (3) 通过人类输入来完成。\n\n思维树（Yao 等，2023）通过探索每一步的多种推理可能性扩展了 CoT。它首先将问题分解为多个思维步骤，并为每一步生成多个思维，创建一个树结构。搜索过程可以是 BFS（广度优先搜索）或 DFS（深度优先搜索），每个状态通过分类器（通过提示）或多数投票进行评估。\n任务分解可以通过 (1) LLM 使用简单提示，如 "步骤为 XYZ.\\n1."，"实现 XYZ 的子目标是什么？"，(2) 使用特定任务的指令；例如 "写一个故事大纲。" 用于写小说，或 (3) 通过人类输入来完成。', name='blog_post_retriever', tool_call_id='call_rg7zKTE5e0ICxVSslJ1u9LMg')]}}
----
{'agent': {'messages': [AIMessage(content='任务分解是一种将复杂任务分解为更小和更简单步骤的技术。这种方法有助于管理和解决复杂问题，通过将其分解为更可管理的组件。通过分解任务，代理或模型可以更好地理解所涉及的步骤并相应地规划其行动。像思维链（CoT）和思维树这样的技术是通过将任务分解为更小步骤来增强模型在复杂任务上的表现的示例。', response_metadata={'token_usage': {'completion_tokens': 87, 'prompt_tokens': 659, 'total_tokens': 746}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-b9166386-83e5-4b82-9a4b-590e5fa76671-0')]}}
----
```
上面，代理在将查询插入工具时，剔除了“什么”和“是”等不必要的词。

这一原则使得代理在必要时能够利用对话的上下文：

```python
query = "What according to the blog post are common ways of doing it? redo the search"

for s in agent_executor.stream(
    {"messages": [HumanMessage(content=query)]}, config=config
):
    print(s)
    print("----")
```
```output
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_6kbxTU5CDWLmF9mrvR7bWSkI', 'function': {'arguments': '{"query":"Common ways of task decomposition"}', 'name': 'blog_post_retriever'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 21, 'prompt_tokens': 769, 'total_tokens': 790}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': None, 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-2d2c8327-35cd-484a-b8fd-52436657c2d8-0', tool_calls=[{'name': 'blog_post_retriever', 'args': {'query': 'Common ways of task decomposition'}, 'id': 'call_6kbxTU5CDWLmF9mrvR7bWSkI'}])]}}
----
``````output
Error in LangChainTracer.on_tool_end callback: TracerException("Found chain run at ID 29553415-e0f4-41a9-8921-ba489e377f68, but expected {'tool'} run.")
``````output
{'tools': {'messages': [ToolMessage(content='Fig. 1. Overview of a LLM-powered autonomous agent system.\nComponent One: Planning#\nA complicated task usually involves many steps. An agent needs to know what they are and plan ahead.\nTask Decomposition#\nChain of thought (CoT; Wei et al. 2022) has become a standard prompting technique for enhancing model performance on complex tasks. The model is instructed to “think step by step” to utilize more test-time computation to decompose hard tasks into smaller and simpler steps. CoT transforms big tasks into multiple manageable tasks and shed lights into an interpretation of the model’s thinking process.\n\nFig. 1. Overview of a LLM-powered autonomous agent system.\nComponent One: Planning#\nA complicated task usually involves many steps. An agent needs to know what they are and plan ahead.\nTask Decomposition#\nChain of thought (CoT; Wei et al. 2022) has become a standard prompting technique for enhancing model performance on complex tasks. The model is instructed to “think step by step” to utilize more test-time computation to decompose hard tasks into smaller and simpler steps. CoT transforms big tasks into multiple manageable tasks and shed lights into an interpretation of the model’s thinking process.\n\nTree of Thoughts (Yao et al. 2023) extends CoT by exploring multiple reasoning possibilities at each step. It first decomposes the problem into multiple thought steps and generates multiple thoughts per step, creating a tree structure. The search process can be BFS (breadth-first search) or DFS (depth-first search) with each state evaluated by a classifier (via a prompt) or majority vote.\nTask decomposition can be done (1) by LLM with simple prompting like "Steps for XYZ.\\n1.", "What are the subgoals for achieving XYZ?", (2) by using task-specific instructions; e.g. "Write a story outline." for writing a novel, or (3) with human inputs.\n\nTree of Thoughts (Yao et al. 2023) extends CoT by exploring multiple reasoning possibilities at each step. It first decomposes the problem into multiple thought steps and generates multiple thoughts per step, creating a tree structure. The search process can be BFS (breadth-first search) or DFS (depth-first search) with each state evaluated by a classifier (via a prompt) or majority vote.\nTask decomposition can be done (1) by LLM with simple prompting like "Steps for XYZ.\\n1.", "What are the subgoals for achieving XYZ?", (2) by using task-specific instructions; e.g. "Write a story outline." for writing a novel, or (3) with human inputs.', name='blog_post_retriever', tool_call_id='call_6kbxTU5CDWLmF9mrvR7bWSkI')]}}
----
{'agent': {'messages': [AIMessage(content='Common ways of task decomposition include:\n1. Using LLM with simple prompting like "Steps for XYZ" or "What are the subgoals for achieving XYZ?"\n2. Using task-specific instructions, for example, "Write a story outline" for writing a novel.\n3. Involving human inputs in the task decomposition process.', response_metadata={'token_usage': {'completion_tokens': 67, 'prompt_tokens': 1339, 'total_tokens': 1406}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-9ad14cde-ca75-4238-a868-f865e0fc50dd-0')]}}
----
```
注意到代理能够推断出我们查询中的“它”指的是“任务分解”，并因此生成了一个合理的搜索查询——在这种情况下，“任务分解的常见方法”。

### 整合步骤

为了方便，我们将所有必要的步骤整合在一个代码单元中：


```python
import bs4
from langchain.tools.retriever import create_retriever_tool
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langgraph.checkpoint.sqlite import SqliteSaver

memory = SqliteSaver.from_conn_string(":memory:")
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)


### 构建检索器 ###
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
retriever = vectorstore.as_retriever()


### 构建检索工具 ###
tool = create_retriever_tool(
    retriever,
    "blog_post_retriever",
    "搜索并返回来自自主代理博客文章的摘录。",
)
tools = [tool]


agent_executor = create_react_agent(llm, tools, checkpointer=memory)
```

## 下一步

我们已经涵盖了构建基本对话问答应用程序的步骤：

- 我们使用链构建了一个可预测的应用程序，该应用程序为每个用户输入生成搜索查询；
- 我们使用代理构建了一个“决定”何时以及如何生成搜索查询的应用程序。

要探索不同类型的检索器和检索策略，请访问如何指南的 [retrievers](/docs/how_to#retrievers) 部分。

有关 LangChain 对话记忆抽象的详细讲解，请访问 [如何添加消息历史（记忆）](/docs/how_to/message_history) LCEL 页面。

要了解更多关于代理的信息，请访问 [代理模块](/docs/tutorials/agents)。