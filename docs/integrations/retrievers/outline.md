---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/outline.ipynb
---

# 大纲

>[Outline](https://www.getoutline.com/) 是一个开源的协作知识库平台，旨在团队信息共享。

本笔记本展示了如何从您的 Outline 实例中检索文档，以便转化为下游使用的文档格式。

## 设置


```python
%pip install --upgrade --quiet langchain langchain-openai
```

您首先需要为您的 Outline 实例 [创建一个 api 密钥](https://www.getoutline.com/developers#section/Authentication)。然后您需要设置以下环境变量：


```python
import os

os.environ["OUTLINE_API_KEY"] = "xxx"
os.environ["OUTLINE_INSTANCE_URL"] = "https://app.getoutline.com"
```

`OutlineRetriever` 有以下参数：
- 可选的 `top_k_results`：默认值为 3。用于限制检索的文档数量。
- 可选的 `load_all_available_meta`：默认值为 False。默认情况下仅检索最重要的字段：`title`、`source`（文档的 URL）。如果为 True，则还会检索其他字段。
- 可选的 `doc_content_chars_max`：默认值为 4000。用于限制每个检索文档的字符数。

`get_relevant_documents()` 有一个参数，`query`：用于在您的 Outline 实例中查找文档的自由文本。

## 示例

### 运行检索器


```python
from langchain_community.retrievers import OutlineRetriever
```


```python
retriever = OutlineRetriever()
```


```python
retriever.invoke("LangChain", doc_content_chars_max=100)
```



```output
[Document(page_content='此指南展示了如何使用针对对话优化的代理。其他代理通常优化为使用工具来找出最佳响应，这在对话环境中并不理想，因为您可能希望代理能够与用户进行聊天。\n\n如果将其与标准的 ReAct 代理进行比较，主要区别在于提示。我们希望它更具对话性。\n\nfrom langchain.agents import AgentType, Tool, initialize_agent\n\nfrom langchain_openai import OpenAI\n\nfrom langchain.memory import ConversationBufferMemory\n\nfrom langchain_community.utilities import SerpAPIWrapper\n\nsearch = SerpAPIWrapper() tools = \\[ Tool( name="Current Search", func=search.run, description="适用于您需要回答有关当前事件或世界现状的问题时", ), \\]\n\n\\\nllm = OpenAI(temperature=0)\n\n使用 LCEL\n\n我们将首先展示如何使用 LCEL 创建此代理\n\nfrom langchain import hub\n\nfrom langchain.agents.format_scratchpad import format_log_to_str\n\nfrom langchain.agents.output_parsers import ReActSingleInputOutputParser\n\nfrom langchain.tools.render import render_text_description\n\nprompt = hub.pull("hwchase17/react-chat")\n\nprompt = prompt.partial( tools=render_text_description(tools), tool_names=", ".join(\\[[t.name](http://t.name) for t in tools\\]), )\n\nllm_with_stop = llm.bind(stop=\\["\\nObservation"\\])\n\nagent = ( { "input": lambda x: x\\["input"\\], "agent_scratchpad": lambda x: format_log_to_str(x\\["intermediate_steps"\\]), "chat_history": lambda x: x\\["chat_history"\\], } | prompt | llm_with_stop | ReActSingleInputOutputParser() )\n\nfrom langchain.agents import AgentExecutor\n\nmemory = ConversationBufferMemory(memory_key="chat_history") agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, memory=memory)\n\nagent_executor.invoke({"input": "hi, i am bob"})\\["output"\\]\n\n```\n> 进入新的 AgentExecutor 链...\n\n思考：我需要使用工具吗？不\n最终答案：嗨 Bob，很高兴见到你！今天我能帮你什么？\n\n> 完成链。\n```\n\n\\\n\'嗨 Bob，很高兴见到你！今天我能帮你什么？\'\n\nagent_executor.invoke({"input": "whats my name?"})\\["output"\\]\n\n```\n> 进入新的 AgentExecutor 链...\n\n思考：我需要使用工具吗？不\n最终答案：你的名字是 Bob。\n\n> 完成链。\n```\n\n\\\n\'你的名字是 Bob。\'\n\nagent_executor.invoke({"input": "what are some movies showing 9/21/2023?"})\\["output"\\]\n\n```\n> 进入新的 AgentExecutor 链...\n\n思考：我需要使用工具吗？是\n行动：当前搜索\n行动输入：2023年9月21日上映的电影[\'2023年9月电影：创作者 • 笨钱 • 续集 • 凶杀室 • 发明者 • 复仇者3 • 小狗巡逻队：强大的电影，...\'] 我需要使用工具吗？不\n最终答案：根据当前搜索，2023年9月21日上映的一些电影是《创作者》、《笨钱》、《续集》、《凶杀室》、《发明者》、《复仇者3》和《小狗巡逻队：强大的电影》。\n\n> 完成链。\n```\n\n\\\n\'根据当前搜索，2023年9月21日上映的一些电影是《创作者》、《笨钱》、《续集》、《凶杀室》、《发明者》、《复仇者3》和《小狗巡逻队：强大的电影》。\'\n\n\\\n使用现成的代理\n\n我们还可以使用现成的代理类创建此代理\n\nagent_executor = initialize_agent( tools, llm, agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION, verbose=True, memory=memory, )\n\n使用聊天模型\n\n我们还可以在这里使用聊天模型。主要区别在于所使用的提示。\n\nfrom langchain import hub\n\nfrom langchain_openai import ChatOpenAI\n\nprompt = hub.pull("hwchase17/react-chat-json") chat_model = ChatOpenAI(temperature=0, model="gpt-4")\n\nprompt = prompt.partial( tools=render_text_description(tools), tool_names=", ".join(\\[[t.name](http://t.name) for t in tools\\]), )\n\nchat_model_with_stop = chat_model.bind(stop=\\["\\nObservation"\\])\n\nfrom langchain.agents.format_scratchpad import format_log_to_messages\n\nfrom langchain.agents.output_parsers import JSONAgentOutputParser\n\n# 我们需要一些额外的引导，或者 c', metadata={'title': '对话', 'source': 'https://d01.getoutline.com/doc/conversational-B5dBkUgQ4b'}),
 Document(page_content='快速入门\n\n在这个快速入门中，我们将向您展示如何：\n\n设置 LangChain、LangSmith 和 LangServe\n\n使用 LangChain 的最基本和常见的组件：提示模板、模型和输出解析器\n\n使用 LangChain 表达语言，LangChain 构建的协议，便于组件链式连接\n\n使用 LangChain 构建一个简单的应用程序\n\n使用 LangSmith 跟踪您的应用程序\n\n使用 LangServe 部署您的应用程序\n\n这要覆盖的内容还不少！让我们深入了解。\n\n设置\n\n安装\n\n要安装 LangChain，请运行：\n\nPip\n\nConda\n\npip install langchain\n\n有关更多详细信息，请参见我们的安装指南。\n\n环境\n\n使用 LangChain 通常需要与一个或多个模型提供者、数据存储、API 等进行集成。在这个例子中，我们将使用 OpenAI 的模型 API。\n\n首先，我们需要安装他们的 Python 包：\n\npip install openai\n\n访问 API 需要一个 API 密钥，您可以通过创建一个帐户并前往此处获取。获得密钥后，我们希望通过运行以下命令将其设置为环境变量：\n\nexport OPENAI_API_KEY="..."\n\n如果您不想设置环境变量，可以在初始化 OpenAI LLM 类时通过 openai_api_key 命名参数直接传递密钥：\n\nfrom langchain_openai import ChatOpenAI\n\nllm = ChatOpenAI(openai_api_key="...")\n\nLangSmith\n\n您用 LangChain 构建的许多应用程序将包含多个步骤和多次 LLM 调用。随着这些应用程序变得越来越复杂，能够检查链或代理内部发生的具体情况变得至关重要。最好的方法是使用 LangSmith。\n\n请注意，LangSmith 不是必需的，但它是有帮助的。如果您确实想使用 LangSmith，在您在上面的链接注册后，请确保设置您的环境变量以开始记录跟踪：\n\nexport LANGCHAIN_TRACING_V2="true" export LANGCHAIN_API_KEY=...\n\nLangServe\n\nLangServe 帮助开发人员将 LangChain 链作为 REST API 部署。您不需要使用 LangServe 来使用 LangChain，但在本指南中，我们将展示如何使用 LangServe 部署您的应用。\n\n使用以下命令安装：\n\npip install "langserve\\[all\\]"\n\n使用 LangChain 构建\n\nLangChain 提供了许多模块，可用于构建语言模型应用程序。模块可以作为独立组件在简单应用中使用，也可以组合用于更复杂的用例。组合由 LangChain 表达语言（LCEL）提供支持，该语言定义了许多模块实现的统一可运行接口，使得可以无缝地连接组件。\n\n最简单和最常见的链包含三样东西：\n\nLLM/聊天模型：语言模型是这里的核心推理引擎。为了与 LangChain 一起工作，您需要了解不同类型的语言模型以及如何与它们一起工作。提示模板：这为语言模型提供指令。这控制了语言模型的输出，因此理解如何构建提示和不同的提示策略至关重要。输出解析器：这些将语言模型的原始响应转换为更易于使用的格式，从而使其在下游使用输出变得简单。在本指南中，我们将分别介绍这三种组件，然后讨论如何将它们组合在一起。理解这些概念将使您能够有效使用和自定义 LangChain 应用程序。大多数 LangChain 应用程序允许您配置模型和/或提示，因此知道如何利用这一点将是一个重要的促进因素。\n\nLLM / 聊天模型\n\n有两种类型的语言模型：\n\nLLM：基础模型接受字符串作为输入并返回字符串\n\n聊天模型：基础模型接受消息列表作为输入并返回消息\n\n字符串很简单，但消息到底是什么？基本消息接口由 BaseMessage 定义，它有两个必需的属性：\n\ncontent：消息的内容。通常是一个字符串。role：BaseMessage 的来源实体。LangChain 提供了几个 ob', metadata={'title': '快速入门', 'source': 'https://d01.getoutline.com/doc/quick-start-jGuGGGOTuL'}),
 Document(page_content='此指南展示了如何使用代理实现 [ReAct](https://react-lm.github.io/) 逻辑。\n\n```javascript\nfrom langchain.agents import AgentType, initialize_agent, load_tools\nfrom langchain_openai import OpenAI\n```\n\n首先，让我们加载将用于控制代理的语言模型。\n\n```javascript\nllm = OpenAI(temperature=0)\n```\n\n接下来，让我们加载一些工具。请注意，llm-math 工具使用 LLM，因此我们需要传递它。\n\n```javascript\ntools = load_tools(["serpapi", "llm-math"], llm=llm)\n```\n\n## 使用 LCEL[\u200b](/docs/modules/agents/agent_types/react#using-lcel "直接链接到使用 LCEL")\n\n我们将首先展示如何使用 LCEL 创建代理\n\n```javascript\nfrom langchain import hub\nfrom langchain.agents.format_scratchpad import format_log_to_str\nfrom langchain.agents.output_parsers import ReActSingleInputOutputParser\nfrom langchain.tools.render import render_text_description\n```\n\n```javascript\nprompt = hub.pull("hwchase17/react")\nprompt = prompt.partial(\n    tools=render_text_description(tools),\n    tool_names=", ".join([t.name for t in tools]),\n)\n```\n\n```javascript\nllm_with_stop = llm.bind(stop=["\\nObservation"])\n```\n\n```javascript\nagent = (\n    {\n        "input": lambda x: x["input"],\n        "agent_scratchpad": lambda x: format_log_to_str(x["intermediate_steps"]),\n    }\n    | prompt\n    | llm_with_stop\n    | ReActSingleInputOutputParser()\n)\n```\n\n```javascript\nfrom langchain.agents import AgentExecutor\n```\n\n```javascript\nagent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)\n```\n\n```javascript\nagent_executor.invoke(\n    {\n        "input": "谁是莱昂纳多·迪卡普里奥的女朋友？她现在的年龄提高到 0.43 的幂？"\n    }\n)\n```\n\n```javascript\n    \n    \n    > 进入新的 AgentExecutor 链...\n     我需要找出莱昂纳多·迪卡普里奥的女朋友是谁，然后计算她的年龄提高到 0.43 的幂。\n    行动：搜索\n    行动输入："莱昂纳多·迪卡普里奥女朋友"model Vittoria Ceretti 我需要找出 Vittoria Ceretti 的年龄\n    行动：搜索\n    行动输入："Vittoria Ceretti 年龄"25岁 我需要计算 25 提高到 0.43 的幂\n    行动：计算器\n    行动输入：25^0.43答案：3.991298452658078 我现在知道最终答案\n    最终答案：莱昂纳多·迪卡普里奥的女朋友是 Vittoria Ceretti，她现在的年龄提高到 0.43 的幂是 3.991298452658078。\n    \n    > 完成链。\n\n\n\n\n\n    {\'input\': "谁是莱昂纳多·迪卡普里奥的女朋友？她现在的年龄提高到 0.43 的幂？",\n     \'output\': "莱昂纳多·迪卡普里奥的女朋友是 Vittoria Ceretti，她现在的年龄提高到 0.43 的幂是 3.991298452658078。"}\n```\n\n## 使用 ZeroShotReactAgent[\u200b](/docs/modules/agents/agent_types/react#using-zeroshotreactagent "直接链接到使用 ZeroShotReactAgent")\n\n我们现在将展示如何使用现成的代理实现\n\n```javascript\nagent_executor = initialize_agent(\n    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True\n)\n```\n\n```javascript\nagent_executor.invoke(\n    {\n        "input": "谁是莱昂纳多·迪卡普里奥的女朋友？她现在的年龄提高到 0.43 的幂？"\n    }\n)\n```\n\n```javascript\n    \n    \n    > 进入新的 AgentExecutor 链...\n     我需要找出莱昂纳多·迪卡普里奥的女朋友是谁，然后计算她的年龄提高到 0.43 的幂。\n    行动：搜索\n    行动输入："莱昂纳多·迪卡普里奥女朋友"\n    观察：model Vittoria Ceretti\n    思考：我需要找出 Vittoria Ceretti 的年龄\n    行动：搜索\n    行动输入："Vittoria Ceretti 年龄"\n    观察：25岁\n    思考：我需要计算 25 提高到 0.43 的幂\n    行动：计算器\n    行动输入：25^0.43\n    观察：答案：3.991298452658078\n    思考：我现在知道最终答案\n    最终答案：莱昂纳多·迪卡普里奥的女朋友是 Vittoria Ceretti，她现在的年龄提高到 0.43 的幂是 3.991298452658078。\n    \n    > 完成链。\n\n\n\n\n\n    {\'input\': "谁是莱昂纳多·迪卡普里奥的女朋友？她现在的年龄提高到 0.43 的幂？",\n     \'output\': "莱昂纳多·迪卡普里奥的女朋友是 Vittoria Ceretti，她现在的年龄提高到 0.43 的幂是 3.991298452658078。"}\n```\n\n## 使用 Outline 文档回答问题\n\n```python\nimport os
```

```python
from getpass import getpass

os.environ["OPENAI_API_KEY"] = getpass("OpenAI API 密钥:")
```


```python
from langchain.chains import ConversationalRetrievalChain
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-3.5-turbo")
qa = ConversationalRetrievalChain.from_llm(model, retriever=retriever)
```


```python
qa({"question": "什么是 langchain？", "chat_history": {}})
```



```output
{'question': '什么是 langchain？',
 'chat_history': {},
 'answer': "LangChain 是一个用于开发由语言模型驱动的应用程序的框架。它提供了一组库和工具，使开发人员能够构建上下文感知和基于推理的应用程序。LangChain 允许您将语言模型连接到各种上下文源，例如提示说明、少量示例和内容，以增强模型的响应。它还支持使用 LangChain 表达语言 (LCEL) 组合多个语言模型组件。此外，LangChain 提供现成的链、模板和集成，便于应用程序开发。LangChain 可以与 LangSmith 结合使用，以调试和监控链，并与 LangServe 一起使用，以将应用程序部署为 REST API。"}
```



## 相关

- 检索器 [概念指南](/docs/concepts/#retrievers)
- 检索器 [操作指南](/docs/how_to/#retrievers)