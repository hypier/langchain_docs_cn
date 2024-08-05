---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/tutorials/qa_chat_history.ipynb
sidebar_position: 2
---

# 对话式 RAG

:::info 前提条件

本指南假设您对以下概念有一定了解：

- [聊天历史](/docs/concepts/#chat-history)
- [聊天模型](/docs/concepts/#chat-models)
- [嵌入](/docs/concepts/#embedding-models)
- [向量存储](/docs/concepts/#vector-stores)
- [检索增强生成](/docs/tutorials/rag/)
- [工具](/docs/concepts/#tools)
- [代理](/docs/concepts/#agents)

:::

在许多问答应用中，我们希望允许用户进行互动对话，这意味着应用程序需要某种形式的“记忆”，以记录过去的问题和答案，并具备将这些信息融入当前思考的逻辑。

在本指南中，我们专注于**添加用于整合历史消息的逻辑。** 有关聊天历史管理的更多细节，请参阅[此处](/docs/how_to/message_history)。

我们将介绍两种方法：

1. 链接，其中我们始终执行检索步骤；
2. 代理，其中我们赋予 LLM 自主决定是否以及如何执行检索步骤（或多个步骤）。

作为外部知识来源，我们将使用 Lilian Weng 的同一篇[LLM 驱动的自主代理](https://lilianweng.github.io/posts/2023-06-23-agent/)博客文章，来自[RAG 教程](/docs/tutorials/rag)。

## 设置

### 依赖项

在本次演示中，我们将使用 OpenAI embeddings 和 Chroma 向量存储，但这里展示的所有内容都适用于任何 [Embeddings](/docs/concepts#embedding-models)、[VectorStore](/docs/concepts#vectorstores) 或 [Retriever](/docs/concepts#retrievers)。

我们将使用以下包：

```python
%%capture --no-stderr
%pip install --upgrade --quiet  langchain langchain-community langchainhub langchain-chroma bs4
```

我们需要设置环境变量 `OPENAI_API_KEY`，这可以直接完成或从 `.env` 文件中加载，如下所示：

```python
import getpass
import os

if not os.environ.get("OPENAI_API_KEY"):
    os.environ["OPENAI_API_KEY"] = getpass.getpass()

# import dotenv

# dotenv.load_dotenv()
```

### LangSmith

您使用 LangChain 构建的许多应用程序将包含多个步骤和多次调用 LLM。随着这些应用程序变得越来越复杂，能够检查您的链或代理内部究竟发生了什么变得至关重要。实现这一点的最佳方法是使用 [LangSmith](https://smith.langchain.com)。

请注意，LangSmith 不是必需的，但它是有帮助的。如果您确实想使用 LangSmith，请在上述链接注册后，确保设置您的环境变量以开始记录跟踪：


```python
os.environ["LANGCHAIN_TRACING_V2"] = "true"
if not os.environ.get("LANGCHAIN_API_KEY"):
    os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## Chains {#chains}

让我们首先回顾一下我们在 [Lilian Weng 的博客文章](https://lilianweng.github.io/posts/2023-06-23-agent/) 中构建的 Q&A 应用，该文章介绍了 [RAG 教程](/docs/tutorials/rag)。

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />


```python
import bs4
from langchain import hub
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

# 1. Load, chunk and index the contents of the blog to create a retriever.
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


# 2. Incorporate the retriever into a question-answering chain.
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
```


```python
response = rag_chain.invoke({"input": "What is Task Decomposition?"})
response["answer"]
```



```output
"Task decomposition involves breaking down complex tasks into smaller and simpler steps to make them more manageable for an agent or model. This process helps in guiding the agent through the various subgoals required to achieve the overall task efficiently. Different techniques like Chain of Thought and Tree of Thoughts can be used to decompose tasks into step-by-step processes, enhancing performance and understanding of the model's thinking process."
```


请注意，我们使用了内置的链构造函数 `create_stuff_documents_chain` 和 `create_retrieval_chain`，因此我们解决方案的基本组成部分是：

1. 检索器；
2. 提示；
3. LLM。

这将简化整合聊天历史的过程。

### 添加聊天历史

我们构建的链条直接使用输入查询来检索相关上下文。但在对话环境中，用户查询可能需要对话上下文才能被理解。例如，考虑以下对话：

> 人类: "什么是任务分解？"
>
> AI: "任务分解涉及将复杂任务分解为更小、更简单的步骤，以便让代理或模型更易于管理。"
>
> 人类: "常见的做法有哪些？"

为了回答第二个问题，我们的系统需要理解“它”指的是“任务分解”。

我们需要更新现有应用的两个方面：

1. **提示**: 更新我们的提示以支持历史消息作为输入。
2. **上下文化问题**: 添加一个子链，获取最新的用户问题，并在聊天历史的上下文中重新表述它。这可以简单地理解为构建一个新的“历史感知”检索器。之前我们有：
   - `query` -> `retriever`  
     现在我们将有：
   - `(query, conversation history)` -> `LLM` -> `rephrased query` -> `retriever`

#### 上下文化问题

首先，我们需要定义一个子链，该链接受历史消息和最新的用户问题，并在其引用历史信息时重新表述问题。

我们将使用一个包含 `MessagesPlaceholder` 变量的提示，名称为 "chat_history"。这使我们能够通过 "chat_history" 输入键将消息列表传递给提示，这些消息将在系统消息之后和包含最新问题的人类消息之前插入。

请注意，我们利用辅助函数 [create_history_aware_retriever](https://api.python.langchain.com/en/latest/chains/langchain.chains.history_aware_retriever.create_history_aware_retriever.html) 来处理这一步，该函数管理 `chat_history` 为空的情况，否则按顺序应用 `prompt | llm | StrOutputParser() | retriever`。

`create_history_aware_retriever` 构建一个接受 `input` 和 `chat_history` 作为输入的链，并具有与检索器相同的输出模式。


```python
from langchain.chains import create_history_aware_retriever
from langchain_core.prompts import MessagesPlaceholder

contextualize_q_system_prompt = (
    "给定聊天历史和最新的用户问题，"
    "该问题可能引用聊天历史中的上下文，"
    "形成一个可以独立理解的问题，"
    "不要回答问题，"
    "如果需要则重新表述，其他情况保持原样。"
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
```

该链将输入查询的重新表述添加到我们的检索器之前，以便检索包含对话上下文。

现在我们可以构建完整的 QA 链。这只需将检索器更新为我们的新 `history_aware_retriever`。

同样，我们将使用 [create_stuff_documents_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.combine_documents.stuff.create_stuff_documents_chain.html) 生成一个 `question_answer_chain`，其输入键为 `context`、`chat_history` 和 `input`——它接受检索到的上下文以及对话历史和查询以生成答案。更详细的解释可以在 [这里](/docs/tutorials/rag/#built-in-chains) 找到。

我们通过 [create_retrieval_chain](https://api.python.langchain.com/en/latest/chains/langchain.chains.retrieval.create_retrieval_chain.html) 构建最终的 `rag_chain`。该链按顺序应用 `history_aware_retriever` 和 `question_answer_chain`，保留中间输出，例如检索到的上下文，以方便使用。它具有输入键 `input` 和 `chat_history`，并在输出中包含 `input`、`chat_history`、`context` 和 `answer`。


```python
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain

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

让我们试试这个。下面我们提出一个问题和一个需要上下文化的后续问题，以返回合理的响应。由于我们的链包括一个 `"chat_history"` 输入，调用者需要管理聊天历史。我们可以通过将输入和输出消息附加到列表来实现这一点：


```python
from langchain_core.messages import AIMessage, HumanMessage

chat_history = []

question = "什么是任务分解？"
ai_msg_1 = rag_chain.invoke({"input": question, "chat_history": chat_history})
chat_history.extend(
    [
        HumanMessage(content=question),
        AIMessage(content=ai_msg_1["answer"]),
    ]
)

second_question = "常见的做法有哪些？"
ai_msg_2 = rag_chain.invoke({"input": second_question, "chat_history": chat_history})

print(ai_msg_2["answer"])
```
```output
任务分解可以通过多种方法实现，例如使用链式思维（CoT）或思维树等技术将复杂任务分解为更小的步骤。常见的方法包括用简单的指令提示模型，如“XYZ的步骤”或特定任务的指令，如“写一个故事大纲”。人类输入也可以有效地指导任务分解过程。
```
:::tip

查看 [LangSmith 跟踪](https://smith.langchain.com/public/243301e4-4cc5-4e52-a6e7-8cfe9208398d/r) 

:::

#### 有状态的聊天历史管理

在这里，我们讨论了如何添加应用逻辑以纳入历史输出，但我们仍在手动更新聊天历史并将其插入每个输入。在一个真实的问答应用中，我们希望有某种方式来持久化聊天历史，并自动插入和更新它。

为此，我们可以使用：

- [BaseChatMessageHistory](https://api.python.langchain.com/en/latest/langchain_api_reference.html#module-langchain.memory): 存储聊天历史。
- [RunnableWithMessageHistory](/docs/how_to/message_history): 一个 LCEL 链和 `BaseChatMessageHistory` 的包装器，处理将聊天历史注入输入并在每次调用后更新它。

有关如何将这些类结合使用以创建有状态对话链的详细说明，请访问 [如何添加消息历史（内存）](/docs/how_to/message_history) LCEL 页面。

下面，我们实现第二种选择的简单示例，其中聊天历史存储在一个简单的字典中。LangChain 与 [Redis](/docs/integrations/memory/redis_chat_message_history/) 和其他技术管理内存集成，以提供更强大的持久性。

`RunnableWithMessageHistory` 的实例为您管理聊天历史。它们接受一个带有键（默认是 `"session_id"`）的配置，该键指定要获取并添加到输入的对话历史，并将输出附加到同一对话历史。以下是一个示例：


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
    {"input": "什么是任务分解？"},
    config={
        "configurable": {"session_id": "abc123"}
    },  # 在 `store` 中构建一个键 "abc123"。
)["answer"]
```



```output
'任务分解涉及将复杂任务分解为更小、更简单的步骤，以便让它们更易于管理。技术如链式思维（CoT）和思维树有助于模型将困难任务分解为多个可管理的子任务。这个过程使代理能够提前规划，有效地处理复杂任务。'
```



```python
conversational_rag_chain.invoke(
    {"input": "常见的做法有哪些？"},
    config={"configurable": {"session_id": "abc123"}},
)["answer"]
```



```output
'任务分解可以通过多种方法实现，例如使用语言模型（LLM）进行简单提示、针对特定任务量身定制的任务特定指令，或结合人类输入将任务分解为更小的组成部分。这些方法有助于指导代理逐步思考，将复杂任务分解为更可管理的子目标。'
```


可以在 `store` 字典中检查对话历史：


```python
for message in store["abc123"].messages:
    if isinstance(message, AIMessage):
        prefix = "AI"
    else:
        prefix = "用户"

    print(f"{prefix}: {message.content}\n")
```
```output
用户: 什么是任务分解？

AI: 任务分解涉及将复杂任务分解为更小、更简单的步骤，以便让它们更易于管理。技术如链式思维（CoT）和思维树有助于模型将困难任务分解为多个可管理的子任务。这个过程使代理能够提前规划，有效地处理复杂任务。

用户: 常见的做法有哪些？

AI: 任务分解可以通过多种方法实现，例如使用语言模型（LLM）进行简单提示、针对特定任务量身定制的任务特定指令，或结合人类输入将任务分解为更小的组成部分。这些方法有助于指导代理逐步思考，将复杂任务分解为更可管理的子目标。
```

### 整合内容

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
    "给定聊天记录和最新的用户问题，"
    "该问题可能引用聊天记录中的上下文，"
    "形成一个独立的问题，该问题可以在没有聊天记录的情况下理解。"
    "请不要回答问题，"
    "如果需要，请重新表述问题，否则按原样返回。"
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
    "您是一个负责问答任务的助手。"
    "使用以下检索到的上下文来回答"
    "问题。如果您不知道答案，请说您"
    "不知道。最多使用三句话，并保持"
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


### 有状态地管理聊天记录 ###
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
'Task decomposition is a technique used to break down complex tasks into smaller and simpler steps. It involves transforming big tasks into multiple manageable tasks to facilitate problem-solving. Different methods like Chain of Thought and Tree of Thoughts can be employed to decompose tasks effectively.'
```



```python
conversational_rag_chain.invoke(
    {"input": "What are common ways of doing it?"},
    config={"configurable": {"session_id": "abc123"}},
)["answer"]
```



```output
'Task decomposition can be achieved through various methods such as using prompting techniques like "Steps for XYZ" or "What are the subgoals for achieving XYZ?", providing task-specific instructions like "Write a story outline," or incorporating human inputs to break down complex tasks into smaller components. These approaches help in organizing thoughts and planning ahead for successful task completion.'
```

## 代理 {#agents}

代理利用LLMs的推理能力在执行过程中做出决策。使用代理可以将检索过程中的一些自主权转移出去。尽管它们的行为比链式结构更难以预测，但在这种情况下它们提供了一些优势：

- 代理直接生成检索器的输入，而不一定需要我们像上面那样明确构建上下文；
- 代理可以为查询执行多个检索步骤，或者完全不执行检索步骤（例如，作为对用户的通用问候的回应）。

### 检索工具

代理可以访问“工具”并管理其执行。在这种情况下，我们将把我们的检索器转换为 LangChain 工具，以便代理使用：

```python
from langchain.tools.retriever import create_retriever_tool

tool = create_retriever_tool(
    retriever,
    "blog_post_retriever",
    "搜索并返回来自自主代理博客文章的摘录。",
)
tools = [tool]
```

工具是 LangChain [可运行的](/docs/concepts#langchain-expression-language-lcel)，并实现了常规接口：

```python
tool.invoke("task decomposition")
```

### 代理构造器

现在我们已经定义了工具和 LLM，我们可以创建代理。我们将使用 [LangGraph](/docs/concepts/#langgraph) 来构建代理。目前我们使用的是一个高级接口来构造代理，但 LangGraph 的一个优点是这个高级接口是由一个低级、高度可控的 API 支持的，以防您想修改代理逻辑。

```python
from langgraph.prebuilt import create_react_agent

agent_executor = create_react_agent(llm, tools)
```

我们现在可以试一试。请注意，到目前为止它是无状态的（我们仍然需要添加内存）。

```python
query = "What is Task Decomposition?"

for s in agent_executor.stream(
    {"messages": [HumanMessage(content=query)]},
):
    print(s)
    print("----")
```

LangGraph 自带持久化功能，因此我们不需要使用 ChatMessageHistory！相反，我们可以直接将检查点传递给我们的 LangGraph 代理。

```python
from langgraph.checkpoint.sqlite import SqliteSaver

memory = SqliteSaver.from_conn_string(":memory:")

agent_executor = create_react_agent(llm, tools, checkpointer=memory)
```

这就是构建对话式 RAG 代理所需的一切。

让我们观察它的行为。请注意，如果我们输入的查询不需要检索步骤，代理不会执行检索：

```python
config = {"configurable": {"thread_id": "abc123"}}

for s in agent_executor.stream(
    {"messages": [HumanMessage(content="Hi! I'm bob")]}, config=config
):
    print(s)
    print("----")
```
```output
{'agent': {'messages': [AIMessage(content='Hello Bob! How can I assist you today?', response_metadata={'token_usage': {'completion_tokens': 11, 'prompt_tokens': 67, 'total_tokens': 78}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-022806f0-eb26-4c87-9132-ed2fcc6c21ea-0')]}}
----
```
进一步地，如果我们输入的查询确实需要检索步骤，代理会生成工具的输入：

```python
query = "What is Task Decomposition?"

for s in agent_executor.stream(
    {"messages": [HumanMessage(content=query)]}, config=config
):
    print(s)
    print("----")
```

在上面，代理没有将我们的查询逐字插入工具，而是去掉了“what”和“is”等不必要的词。

同样的原则使代理在必要时能够利用对话的上下文：

```python
query = "What according to the blog post are common ways of doing it? redo the search"

for s in agent_executor.stream(
    {"messages": [HumanMessage(content=query)]}, config=config
):
    print(s)
    print("----")
```

请注意，代理能够推断出我们查询中的“it”指的是“任务分解”，并因此生成了一个合理的搜索查询——在这种情况下是“任务分解的常见方法”。

### 综合起来

为了方便，我们将所有必要的步骤整合在一个代码单元中：

```python
import bs4
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain.tools.retriever import create_retriever_tool
from langchain_chroma import Chroma
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.chat_history import BaseChatMessageHistory
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langgraph.checkpoint.sqlite import SqliteSaver
from langgraph.prebuilt import create_react_agent

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

我们已经介绍了构建基本对话问答应用程序的步骤：

- 我们使用链构建了一个可预测的应用程序，为每个用户输入生成搜索查询；
- 我们使用代理构建了一个“决定”何时以及如何生成搜索查询的应用程序。

要探索不同类型的检索器和检索策略，请访问[检索器](/docs/how_to/#retrievers)部分的操作指南。

有关LangChain对话记忆抽象的详细讲解，请访问[如何添加消息历史（记忆）](/docs/how_to/message_history) LCEL页面。

要了解更多关于代理的信息，请前往[代理模块](/docs/tutorials/agents)。