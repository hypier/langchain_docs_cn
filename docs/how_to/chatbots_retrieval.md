---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/chatbots_retrieval.ipynb
sidebar_position: 2
---

# 如何为聊天机器人添加检索功能

检索是聊天机器人用来增强其响应的常见技术，利用的是聊天模型训练数据之外的数据。本节将介绍如何在聊天机器人的上下文中实现检索，但值得注意的是，检索是一个非常微妙和深奥的话题——我们鼓励您探索[文档的其他部分](/docs/how_to#qa-with-rag)，以获得更深入的了解！

## 设置

您需要安装一些软件包，并将您的 OpenAI API 密钥设置为名为 `OPENAI_API_KEY` 的环境变量：


```python
%pip install -qU langchain langchain-openai langchain-chroma beautifulsoup4

# 设置环境变量 OPENAI_API_KEY 或从 .env 文件加载：
import dotenv

dotenv.load_dotenv()
```
```output
[33mWARNING: You are using pip version 22.0.4; however, version 23.3.2 is available.
You should consider upgrading via the '/Users/jacoblee/.pyenv/versions/3.10.5/bin/python -m pip install --upgrade pip' command.[0m[33m
[0mNote: you may need to restart the kernel to use updated packages.
```


```output
True
```


让我们还设置一个聊天模型，以便在下面的示例中使用。


```python
from langchain_openai import ChatOpenAI

chat = ChatOpenAI(model="gpt-3.5-turbo-1106", temperature=0.2)
```

## 创建检索器

我们将使用 [LangSmith 文档](https://docs.smith.langchain.com/overview) 作为源材料，并将内容存储在向量数据库中以便后续检索。请注意，这个示例将略过一些关于解析和存储数据源的细节 - 你可以在这里查看更多 [关于创建检索系统的深入文档](/docs/how_to#qa-with-rag)。

让我们使用文档加载器从文档中提取文本：

```python
from langchain_community.document_loaders import WebBaseLoader

loader = WebBaseLoader("https://docs.smith.langchain.com/overview")
data = loader.load()
```

接下来，我们将其拆分为更小的块，以便 LLM 的上下文窗口可以处理，并将其存储在向量数据库中：

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=0)
all_splits = text_splitter.split_documents(data)
```

然后，我们将这些块嵌入并存储在向量数据库中：

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings

vectorstore = Chroma.from_documents(documents=all_splits, embedding=OpenAIEmbeddings())
```

最后，让我们从初始化的向量数据库中创建一个检索器：

```python
# k 是要检索的块数
retriever = vectorstore.as_retriever(k=4)

docs = retriever.invoke("Can LangSmith help test my LLM applications?")

docs
```

```output
[Document(page_content='Skip to main content🦜️🛠️ LangSmith DocsPython DocsJS/TS DocsSearchGo to AppLangSmithOverviewTracingTesting & EvaluationOrganizationsHubLangSmith CookbookOverviewOn this pageLangSmith Overview and User GuideBuilding reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.Over the past two months, we at LangChain', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
 Document(page_content='LangSmith Overview and User Guide | 🦜️🛠️ LangSmith', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
 Document(page_content='You can also quickly edit examples and add them to datasets to expand the surface area of your evaluation sets or to fine-tune a model for improved quality or reduced costs.Monitoring\u200bAfter all this, your app might finally ready to go in production. LangSmith can also be used to monitor your application in much the same way that you used for debugging. You can log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise. Each run can also be', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
 Document(page_content="does that affect the output?\u200bSo you notice a bad output, and you go into LangSmith to see what's going on. You find the faulty LLM call and are now looking at the exact input. You want to try changing a word or a phrase to see what happens -- what do you do?We constantly ran into this issue. Initially, we copied the prompt to a playground of sorts. But this got annoying, so we built a playground of our own! When examining an LLM call, you can click the Open in Playground button to access this", metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'})]
```

我们可以看到，上述检索器的调用结果包含了一些 LangSmith 文档中的部分内容，这些内容包含了我们的聊天机器人在回答问题时可以使用的测试信息。现在我们已经拥有一个可以从 LangSmith 文档中返回相关数据的检索器！

## 文档链

现在我们有一个可以返回 LangChain 文档的检索器，让我们创建一个可以使用这些文档作为上下文来回答问题的链。我们将使用 `create_stuff_documents_chain` 辅助函数将所有输入文档“填充”到提示中。它还将处理将文档格式化为字符串。

除了聊天模型外，该函数还期望一个包含 `context` 变量的提示，以及一个名为 `messages` 的聊天历史消息占位符。我们将创建一个合适的提示并按如下所示传递它：

```python
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

SYSTEM_TEMPLATE = """
Answer the user's questions based on the below context. 
If the context doesn't contain any relevant information to the question, don't make something up and just say "I don't know":

<context>
{context}
</context>
"""

question_answering_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            SYSTEM_TEMPLATE,
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)

document_chain = create_stuff_documents_chain(chat, question_answering_prompt)
```

我们可以单独调用这个 `document_chain` 来回答问题。让我们使用上面检索到的文档和相同的问题 `langsmith 如何帮助测试？`：

```python
from langchain_core.messages import HumanMessage

document_chain.invoke(
    {
        "context": docs,
        "messages": [
            HumanMessage(content="Can LangSmith help test my LLM applications?")
        ],
    }
)
```

```output
'Yes, LangSmith can help test and evaluate your LLM applications. It simplifies the initial setup, and you can use it to monitor your application, log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise.'
```

看起来不错！为了比较，我们可以尝试在没有上下文文档的情况下进行，并比较结果：

```python
document_chain.invoke(
    {
        "context": [],
        "messages": [
            HumanMessage(content="Can LangSmith help test my LLM applications?")
        ],
    }
)
```

```output
"I don't know about LangSmith's specific capabilities for testing LLM applications. It's best to reach out to LangSmith directly to inquire about their services and how they can assist with testing your LLM applications."
```

我们可以看到 LLM 没有返回任何结果。

## 检索链

让我们将这个文档链与检索器结合起来。这是一个实现方式：

```python
from typing import Dict

from langchain_core.runnables import RunnablePassthrough


def parse_retriever_input(params: Dict):
    return params["messages"][-1].content


retrieval_chain = RunnablePassthrough.assign(
    context=parse_retriever_input | retriever,
).assign(
    answer=document_chain,
)
```

给定一系列输入消息，我们提取列表中最后一条消息的内容，并将其传递给检索器以获取一些文档。然后，我们将这些文档作为上下文传递给我们的文档链，以生成最终的响应。

调用这个链结合了上述两个步骤：

```python
retrieval_chain.invoke(
    {
        "messages": [
            HumanMessage(content="Can LangSmith help test my LLM applications?")
        ],
    }
)
```

```output
{'messages': [HumanMessage(content='Can LangSmith help test my LLM applications?')],
 'context': [Document(page_content='Skip to main content🦜️🛠️ LangSmith DocsPython DocsJS/TS DocsSearchGo to AppLangSmithOverviewTracingTesting & EvaluationOrganizationsHubLangSmith CookbookOverviewOn this pageLangSmith Overview and User GuideBuilding reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.Over the past two months, we at LangChain', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
  Document(page_content='LangSmith Overview and User Guide | 🦜️🛠️ LangSmith', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
  Document(page_content='You can also quickly edit examples and add them to datasets to expand the surface area of your evaluation sets or to fine-tune a model for improved quality or reduced costs.Monitoring\u200bAfter all this, your app might finally ready to go in production. LangSmith can also be used to monitor your application in much the same way that you used for debugging. You can log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise. Each run can also be', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
  Document(page_content="does that affect the output?\u200bSo you notice a bad output, and you go into LangSmith to see what's going on. You find the faulty LLM call and are now looking at the exact input. You want to try changing a word or a phrase to see what happens -- what do you do?We constantly ran into this issue. Initially, we copied the prompt to a playground of sorts. But this got annoying, so we built a playground of our own! When examining an LLM call, you can click the Open in Playground button to access this", metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'})],
 'answer': '是的，LangSmith可以帮助测试和评估您的LLM应用程序。它简化了初始设置，您可以使用它监控您的应用程序，记录所有痕迹，可视化延迟和令牌使用统计信息，并在出现特定问题时进行故障排除。'}
```

## 查询转换

我们的检索链能够回答有关 LangSmith 的问题，但存在一个问题——聊天机器人与用户的互动是对话式的，因此必须处理后续问题。

当前形式的链在这方面会遇到困难。考虑一个对我们原始问题的后续问题，例如 `Tell me more!`。如果我们直接用这个查询调用我们的检索器，我们会得到与 LLM 应用测试无关的文档：

```python
retriever.invoke("Tell me more!")
```

```output
[Document(page_content='You can also quickly edit examples and add them to datasets to expand the surface area of your evaluation sets or to fine-tune a model for improved quality or reduced costs.Monitoring\u200bAfter all this, your app might finally ready to go in production. LangSmith can also be used to monitor your application in much the same way that you used for debugging. You can log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise. Each run can also be', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
 Document(page_content='playground. Here, you can modify the prompt and re-run it to observe the resulting changes to the output - as many times as needed!Currently, this feature supports only OpenAI and Anthropic models and works for LLM and Chat Model calls. We plan to extend its functionality to more LLM types, chains, agents, and retrievers in the future.What is the exact sequence of events?\u200bIn complicated chains and agents, it can often be hard to understand what is going on under the hood. What calls are being', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
 Document(page_content='however, there is still no complete substitute for human review to get the utmost quality and reliability from your application.', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
 Document(page_content='Skip to main content🦜️🛠️ LangSmith DocsPython DocsJS/TS DocsSearchGo to AppLangSmithOverviewTracingTesting & EvaluationOrganizationsHubLangSmith CookbookOverviewOn this pageLangSmith Overview and User GuideBuilding reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.Over the past two months, we at LangChain', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'})]
```

这是因为检索器没有固有的状态概念，只会提取与给定查询最相似的文档。为了解决这个问题，我们可以将查询转换为独立的查询，而不依赖于 LLM 的任何外部引用。

这是一个示例：

```python
from langchain_core.messages import AIMessage, HumanMessage

query_transform_prompt = ChatPromptTemplate.from_messages(
    [
        MessagesPlaceholder(variable_name="messages"),
        (
            "user",
            "根据上述对话生成一个搜索查询，以查找与对话相关的信息。仅回复查询，不要其他内容。",
        ),
    ]
)

query_transformation_chain = query_transform_prompt | chat

query_transformation_chain.invoke(
    {
        "messages": [
            HumanMessage(content="Can LangSmith help test my LLM applications?"),
            AIMessage(
                content="Yes, LangSmith can help test and evaluate your LLM applications. It allows you to quickly edit examples and add them to datasets to expand the surface area of your evaluation sets or to fine-tune a model for improved quality or reduced costs. Additionally, LangSmith can be used to monitor your application, log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise."
            ),
            HumanMessage(content="Tell me more!"),
        ],
    }
)
```

```output
AIMessage(content='"LangSmith LLM application testing and evaluation"')
```

太棒了！这个转换后的查询将拉取与 LLM 应用测试相关的上下文文档。

让我们将其添加到我们的检索链中。我们可以如下包装我们的检索器：

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnableBranch

query_transforming_retriever_chain = RunnableBranch(
    (
        lambda x: len(x.get("messages", [])) == 1,
        # 如果只有一条消息，则将该消息的内容传递给检索器
        (lambda x: x["messages"][-1].content) | retriever,
    ),
    # 如果有消息，则将输入传递给 LLM 链以转换查询，然后传递给检索器
    query_transform_prompt | chat | StrOutputParser() | retriever,
).with_config(run_name="chat_retriever_chain")
```

然后，我们可以使用这个查询转换链使我们的检索链更好地处理此类后续问题：

```python
SYSTEM_TEMPLATE = """
根据以下上下文回答用户的问题。 
如果上下文中没有相关信息，请不要编造，只需说“我不知道”：

<context>
{context}
</context>
"""

question_answering_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            SYSTEM_TEMPLATE,
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)

document_chain = create_stuff_documents_chain(chat, question_answering_prompt)

conversational_retrieval_chain = RunnablePassthrough.assign(
    context=query_transforming_retriever_chain,
).assign(
    answer=document_chain,
)
```

太棒了！让我们用与之前相同的输入调用这个新链：

```python
conversational_retrieval_chain.invoke(
    {
        "messages": [
            HumanMessage(content="Can LangSmith help test my LLM applications?"),
        ]
    }
)
```

```output
{'messages': [HumanMessage(content='Can LangSmith help test my LLM applications?')],
 'context': [Document(page_content='Skip to main content🦜️🛠️ LangSmith DocsPython DocsJS/TS DocsSearchGo to AppLangSmithOverviewTracingTesting & EvaluationOrganizationsHubLangSmith CookbookOverviewOn this pageLangSmith Overview and User GuideBuilding reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.Over the past two months, we at LangChain', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
  Document(page_content='LangSmith Overview and User Guide | 🦜️🛠️ LangSmith', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
  Document(page_content='You can also quickly edit examples and add them to datasets to expand the surface area of your evaluation sets or to fine-tune a model for improved quality or reduced costs.Monitoring\u200bAfter all this, your app might finally ready to go in production. LangSmith can also be used to monitor your application in much the same way that you used for debugging. You can log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise. Each run can also be', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}),
  Document(page_content="does that affect the output?\u200bSo you notice a bad output, and you go into LangSmith to see what's going on. You find the faulty LLM call and are now looking at the exact input. You want to try changing a word or a phrase to see what happens -- what do you do?We constantly ran into this issue. Initially, we copied the prompt to a playground of sorts. But this got annoying, so we built a playground of our own! When examining an LLM call, you can click the Open in Playground button to access this", metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'})],
 'answer': 'Yes, LangSmith can help test and evaluate LLM (Language Model) applications. It simplifies the initial setup, and you can use it to monitor your application, log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise.'}
```

```python
conversational_retrieval_chain.invoke(
    {
        "messages": [
            HumanMessage(content="LangSmith能帮助测试我的LLM应用吗？"),
            AIMessage(
                content="是的，LangSmith可以帮助测试和评估您的LLM应用。它允许您快速编辑示例并将其添加到数据集中，以扩大评估集的覆盖范围，或对模型进行微调以提高质量或降低成本。此外，LangSmith还可以用于监控您的应用，记录所有痕迹，可视化延迟和令牌使用统计信息，并在特定问题出现时进行故障排除。"
            ),
            HumanMessage(content="告诉我更多！"),
        ],
    }
)
```



```output
{'messages': [HumanMessage(content='LangSmith能帮助测试我的LLM应用吗？'),
  AIMessage(content='是的，LangSmith可以帮助测试和评估您的LLM应用。它允许您快速编辑示例并将其添加到数据集中，以扩大评估集的覆盖范围，或对模型进行微调以提高质量或降低成本。此外，LangSmith还可以用于监控您的应用，记录所有痕迹，可视化延迟和令牌使用统计信息，并在特定问题出现时进行故障排除。'),
  HumanMessage(content='告诉我更多！')],
 'context': [Document(page_content='LangSmith概述和用户指南 | 🦜️🛠️ LangSmith', metadata={'description': '构建可靠的LLM应用可能具有挑战性。LangChain简化了初始设置，但仍然需要工作来提高提示、链和代理的性能，使其足够可靠以用于生产。', 'language': 'zh', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith概述和用户指南 | 🦜️🛠️ LangSmith'}),
  Document(page_content='您还可以快速编辑示例并将其添加到数据集中，以扩大评估集的覆盖范围，或对模型进行微调以提高质量或降低成本。监控​在经历了这一切之后，您的应用可能终于准备好投入生产。LangSmith还可以用于以与调试相同的方式监控您的应用。您可以记录所有痕迹，可视化延迟和令牌使用统计信息，并在特定问题出现时进行故障排除。每次运行也可以', metadata={'description': '构建可靠的LLM应用可能具有挑战性。LangChain简化了初始设置，但仍然需要工作来提高提示、链和代理的性能，使其足够可靠以用于生产。', 'language': 'zh', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith概述和用户指南 | 🦜️🛠️ LangSmith'}),
  Document(page_content='跳转到主要内容🦜️🛠️ LangSmith文档Python文档JS/TS文档搜索前往应用LangSmith概述追踪测试与评估组织中心LangSmith食谱概述在此页面LangSmith概述和用户指南构建可靠的LLM应用可能具有挑战性。LangChain简化了初始设置，但仍然需要工作来提高提示、链和代理的性能，使其足够可靠以用于生产。在过去的两个月里，我们在LangChain', metadata={'description': '构建可靠的LLM应用可能具有挑战性。LangChain简化了初始设置，但仍然需要工作来提高提示、链和代理的性能，使其足够可靠以用于生产。', 'language': 'zh', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith概述和用户指南 | 🦜️🛠️ LangSmith'}),
  Document(page_content='LangSmith使手动审核和注释运行变得简单，通过注释队列。这些队列允许您根据模型类型或自动评估分数等标准选择任何运行，并将其排队进行人工审核。作为审核员，您可以快速浏览运行，查看输入、输出和任何现有标签，然后添加自己的反馈。我们通常出于几个原因使用这个功能：评估自动评估者难以处理的主观质量，例如', metadata={'description': '构建可靠的LLM应用可能具有挑战性。LangChain简化了初始设置，但仍然需要工作来提高提示、链和代理的性能，使其足够可靠以用于生产。', 'language': 'zh', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith概述和用户指南 | 🦜️🛠️ LangSmith'})],
 'answer': 'LangSmith简化了构建可靠的LLM应用的初始设置，但它承认仍然需要工作来提高提示、链和代理的性能，使其足够可靠以用于生产。它还提供了通过注释队列手动审核和注释运行的能力，允许您根据模型类型或自动评估分数等标准选择运行进行人工审核。此功能对于评估自动评估者难以处理的主观质量特别有用。'}
```


您可以查看[这个LangSmith追踪](https://smith.langchain.com/public/bb329a3b-e92a-4063-ad78-43f720fbb5a2/r)以亲自了解内部查询转换步骤。

## 流式处理

由于该链是用 LCEL 构建的，因此您可以使用熟悉的方法，例如 `.stream()`：


```python
stream = conversational_retrieval_chain.stream(
    {
        "messages": [
            HumanMessage(content="Can LangSmith help test my LLM applications?"),
            AIMessage(
                content="Yes, LangSmith can help test and evaluate your LLM applications. It allows you to quickly edit examples and add them to datasets to expand the surface area of your evaluation sets or to fine-tune a model for improved quality or reduced costs. Additionally, LangSmith can be used to monitor your application, log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise."
            ),
            HumanMessage(content="Tell me more!"),
        ],
    }
)

for chunk in stream:
    print(chunk)
```
```output
{'messages': [HumanMessage(content='Can LangSmith help test my LLM applications?'), AIMessage(content='Yes, LangSmith can help test and evaluate your LLM applications. It allows you to quickly edit examples and add them to datasets to expand the surface area of your evaluation sets or to fine-tune a model for improved quality or reduced costs. Additionally, LangSmith can be used to monitor your application, log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise.'), HumanMessage(content='Tell me more!')]}
{'context': [Document(page_content='LangSmith Overview and User Guide | 🦜️🛠️ LangSmith', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}), Document(page_content='You can also quickly edit examples and add them to datasets to expand the surface area of your evaluation sets or to fine-tune a model for improved quality or reduced costs.Monitoring\u200bAfter all this, your app might finally ready to go in production. LangSmith can also be used to monitor your application in much the same way that you used for debugging. You can log all traces, visualize latency and token usage statistics, and troubleshoot specific issues as they arise. Each run can also be', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}), Document(page_content='Skip to main content🦜️🛠️ LangSmith DocsPython DocsJS/TS DocsSearchGo to AppLangSmithOverviewTracingTesting & EvaluationOrganizationsHubLangSmith CookbookOverviewOn this pageLangSmith Overview and User GuideBuilding reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.Over the past two months, we at LangChain', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'}), Document(page_content='LangSmith makes it easy to manually review and annotate runs through annotation queues.These queues allow you to select any runs based on criteria like model type or automatic evaluation scores, and queue them up for human review. As a reviewer, you can then quickly step through the runs, viewing the input, output, and any existing tags before adding your own feedback.We often use this for a couple of reasons:To assess subjective qualities that automatic evaluators struggle with, like', metadata={'description': 'Building reliable LLM applications can be challenging. LangChain simplifies the initial setup, but there is still work needed to bring the performance of prompts, chains and agents up the level where they are reliable enough to be used in production.', 'language': 'en', 'source': 'https://docs.smith.langchain.com/overview', 'title': 'LangSmith Overview and User Guide | 🦜️🛠️ LangSmith'})]}
{'answer': ''}
{'answer': 'Lang'}
{'answer': 'Smith'}
{'answer': ' simpl'}
{'answer': 'ifies'}
{'answer': ' the'}
{'answer': ' initial'}
{'answer': ' setup'}
{'answer': ' for'}
{'answer': ' building'}
{'answer': ' reliable'}
{'answer': ' L'}
{'answer': 'LM'}
{'answer': ' applications'}
{'answer': '.'}
{'answer': ' It'}
{'answer': ' provides'}
{'answer': ' features'}
{'answer': ' for'}
{'answer': ' manually'}
{'answer': ' reviewing'}
{'answer': ' and'}
{'answer': ' annot'}
{'answer': 'ating'}
{'answer': ' runs'}
{'answer': ' through'}
{'answer': ' annotation'}
{'answer': ' queues'}
{'answer': ','}
{'answer': ' allowing'}
{'answer': ' you'}
{'answer': ' to'}
{'answer': ' select'}
{'answer': ' runs'}
{'answer': ' based'}
{'answer': ' on'}
{'answer': ' criteria'}
{'answer': ' like'}
{'answer': ' model'}
{'answer': ' type'}
{'answer': ' or'}
{'answer': ' automatic'}
{'answer': ' evaluation'}
{'answer': ' scores'}
{'answer': ','}
{'answer': ' and'}
{'answer': ' queue'}
{'answer': ' them'}
{'answer': ' up'}
{'answer': ' for'}
{'answer': ' human'}
{'answer': ' review'}
{'answer': '.'}
{'answer': ' As'}
{'answer': ' a'}
{'answer': ' reviewer'}
{'answer': ','}
{'answer': ' you'}
{'answer': ' can'}
{'answer': ' quickly'}
{'answer': ' step'}
{'answer': ' through'}
{'answer': ' the'}
{'answer': ' runs'}
{'answer': ','}
{'answer': ' view'}
{'answer': ' the'}
{'answer': ' input'}
{'answer': ','}
{'answer': ' output'}
{'answer': ','}
{'answer': ' and'}
{'answer': ' any'}
{'answer': ' existing'}
{'answer': ' tags'}
{'answer': ' before'}
{'answer': ' adding'}
{'answer': ' your'}
{'answer': ' own'}
{'answer': ' feedback'}
{'answer': '.'}
{'answer': ' This'}
{'answer': ' can'}
{'answer': ' be'}
{'answer': ' particularly'}
{'answer': ' useful'}
{'answer': ' for'}
{'answer': ' assessing'}
{'answer': ' subjective'}
{'answer': ' qualities'}
{'answer': ' that'}
{'answer': ' automatic'}
{'answer': ' evalu'}
{'answer': 'ators'}
{'answer': ' struggle'}
{'answer': ' with'}
{'answer': '.'}
{'answer': ''}
```

## 进一步阅读

本指南仅仅触及了检索技术的表面。有关获取、准备和检索最相关数据的更多不同方法，请查看相关的操作指南 [这里](/docs/how_to#document-loaders)。