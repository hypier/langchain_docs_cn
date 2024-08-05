---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/routing.ipynb
sidebar_position: 3
keywords: [RunnableBranch, LCEL]
---

# 如何在子链之间进行路由

:::info 前提条件

本指南假设您熟悉以下概念：
- [LangChain 表达式语言 (LCEL)](/docs/concepts/#langchain-expression-language)
- [链式可运行对象](/docs/how_to/sequence/)
- [运行时配置链参数](/docs/how_to/configure)
- [提示模板](/docs/concepts/#prompt-templates)
- [聊天消息](/docs/concepts/#message-types)

:::

路由允许您创建非确定性的链，其中前一步的输出定义了下一步。路由可以通过允许您定义状态并使用与这些状态相关的信息作为模型调用的上下文，帮助提供与模型交互的结构和一致性。

有两种方法可以执行路由：

1. 有条件地从 [`RunnableLambda`](/docs/how_to/functions) 返回可运行对象（推荐）
2. 使用 `RunnableBranch`（遗留方法）

我们将通过一个两步序列来说明这两种方法，其中第一步将输入问题分类为 `LangChain`、`Anthropic` 或 `Other`，然后路由到相应的提示链。

## 示例设置
首先，让我们创建一个链，用于识别传入的问题是关于 `LangChain`、`Anthropic` 还是 `Other`：


```python
from langchain_anthropic import ChatAnthropic
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate

chain = (
    PromptTemplate.from_template(
        """Given the user question below, classify it as either being about `LangChain`, `Anthropic`, or `Other`.

Do not respond with more than one word.

<question>
{question}
</question>

Classification:"""
    )
    | ChatAnthropic(model_name="claude-3-haiku-20240307")
    | StrOutputParser()
)

chain.invoke({"question": "how do I call Anthropic?"})
```



```output
'Anthropic'
```


现在，让我们创建三个子链：


```python
langchain_chain = PromptTemplate.from_template(
    """You are an expert in langchain. \
Always answer questions starting with "As Harrison Chase told me". \
Respond to the following question:

Question: {question}
Answer:"""
) | ChatAnthropic(model_name="claude-3-haiku-20240307")
anthropic_chain = PromptTemplate.from_template(
    """You are an expert in anthropic. \
Always answer questions starting with "As Dario Amodei told me". \
Respond to the following question:

Question: {question}
Answer:"""
) | ChatAnthropic(model_name="claude-3-haiku-20240307")
general_chain = PromptTemplate.from_template(
    """Respond to the following question:

Question: {question}
Answer:"""
) | ChatAnthropic(model_name="claude-3-haiku-20240307")
```

## 使用自定义函数（推荐）

您还可以使用自定义函数在不同输出之间进行路由。以下是一个示例：

```python
def route(info):
    if "anthropic" in info["topic"].lower():
        return anthropic_chain
    elif "langchain" in info["topic"].lower():
        return langchain_chain
    else:
        return general_chain
```

```python
from langchain_core.runnables import RunnableLambda

full_chain = {"topic": chain, "question": lambda x: x["question"]} | RunnableLambda(
    route
)
```

```python
full_chain.invoke({"question": "如何使用Anthropic？"})
```

```output
AIMessage(content="正如Dario Amodei告诉我的，要使用Anthropic，您可以从探索该公司的官方网站开始，了解他们的使命、价值观以及他们提供的不同服务和产品。Anthropic专注于开发安全和伦理的AI系统，因此他们非常重视透明度和负责任的AI开发。\n\n根据您的具体需求，您可以查看Anthropic的AI研究和开发服务，涵盖自然语言处理、计算机视觉和强化学习等领域。他们还提供咨询和顾问服务，以帮助组织应对AI集成的挑战和机遇。\n\n此外，Anthropic还发布了一些开源AI模型和工具，您可以探索和实验。这些可以是获得Anthropic AI开发方法的实践经验的好方法。\n\n总的来说，Anthropic旨在成为AI领域可靠和值得信赖的合作伙伴，因此我鼓励您直接与他们联系，讨论他们如何最好地支持您的具体需求。", response_metadata={'id': 'msg_01CtLFgFSwvTaJomrihE87Ra', 'content': [ContentBlock(text="正如Dario Amodei告诉我的，要使用Anthropic，您可以从探索该公司的官方网站开始，了解他们的使命、价值观以及他们提供的不同服务和产品。Anthropic专注于开发安全和伦理的AI系统，因此他们非常重视透明度和负责任的AI开发。\n\n根据您的具体需求，您可以查看Anthropic的AI研究和开发服务，涵盖自然语言处理、计算机视觉和强化学习等领域。他们还提供咨询和顾问服务，以帮助组织应对AI集成的挑战和机遇。\n\n此外，Anthropic还发布了一些开源AI模型和工具，您可以探索和实验。这些可以是获得Anthropic AI开发方法的实践经验的好方法。\n\n总的来说，Anthropic旨在成为AI领域可靠和值得信赖的合作伙伴，因此我鼓励您直接与他们联系，讨论他们如何最好地支持您的具体需求。", type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=53, output_tokens=219)})
```

```python
full_chain.invoke({"question": "如何使用LangChain？"})
```

```output
AIMessage(content="正如Harrison Chase告诉我的，使用LangChain涉及几个关键步骤：\n\n1. **设置您的环境**：安装必要的Python包，包括LangChain库本身，以及您的应用程序可能需要的其他依赖项，例如语言模型或其他集成。\n\n2. **了解核心概念**：LangChain围绕几个核心概念展开，如代理、链和工具。熟悉这些概念及其如何协同工作以构建强大的基于语言的应用程序。\n\n3. **确定您的用例**：确定您想使用LangChain构建的任务或应用程序类型，例如聊天机器人、问答系统或文档摘要工具。\n\n4. **选择适当的组件**：根据您的用例，选择合适的LangChain组件，如代理、链和工具，以构建您的应用程序。\n\n5. **与语言模型集成**：LangChain旨在与各种语言模型无缝协作，例如OpenAI的GPT-3或Anthropic的模型。将您选择的语言模型连接到您的LangChain应用程序。\n\n6. **实现您的应用程序逻辑**：使用LangChain的构建块来实现应用程序的特定功能，例如提示语言模型、处理响应以及与其他服务或数据源集成。\n\n7. **测试和迭代**：全面测试您的应用程序，收集反馈，并对设计和实现进行迭代，以提高其性能和用户体验。\n\n正如Harrison Chase强调的，LangChain提供了一个灵活且强大的框架，用于构建基于语言的应用程序，使得更容易利用现代语言模型的能力。通过遵循这些步骤，您可以开始使用LangChain，并创建针对您特定需求的创新解决方案。", response_metadata={'id': 'msg_01H3UXAAHG4TwxJLpxwuuVU7', 'content': [ContentBlock(text="正如Harrison Chase告诉我的，使用LangChain涉及几个关键步骤：\n\n1. **设置您的环境**：安装必要的Python包，包括LangChain库本身，以及您的应用程序可能需要的其他依赖项，例如语言模型或其他集成。\n\n2. **了解核心概念**：LangChain围绕几个核心概念展开，如代理、链和工具。熟悉这些概念及其如何协同工作以构建强大的基于语言的应用程序。\n\n3. **确定您的用例**：确定您想使用LangChain构建的任务或应用程序类型，例如聊天机器人、问答系统或文档摘要工具。\n\n4. **选择适当的组件**：根据您的用例，选择合适的LangChain组件，如代理、链和工具，以构建您的应用程序。\n\n5. **与语言模型集成**：LangChain旨在与各种语言模型无缝协作，例如OpenAI的GPT-3或Anthropic的模型。将您选择的语言模型连接到您的LangChain应用程序。\n\n6. **实现您的应用程序逻辑**：使用LangChain的构建块来实现应用程序的特定功能，例如提示语言模型、处理响应以及与其他服务或数据源集成。\n\n7. **测试和迭代**：全面测试您的应用程序，收集反馈，并对设计和实现进行迭代，以提高其性能和用户体验。\n\n正如Harrison Chase强调的，LangChain提供了一个灵活且强大的框架，用于构建基于语言的应用程序，使得更容易利用现代语言模型的能力。通过遵循这些步骤，您可以开始使用LangChain，并创建针对您特定需求的创新解决方案。", type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=50, output_tokens=400)})
```

```python
full_chain.invoke({"question": "2 + 2等于多少？"})
```

```output
AIMessage(content='4', response_metadata={'id': 'msg_01UAKP81jTZu9fyiyFYhsbHc', 'content': [ContentBlock(text='4', type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=28, output_tokens=5)})
```

## 使用 RunnableBranch

`RunnableBranch` 是一种特殊类型的可运行对象，它允许您根据输入定义一组条件和可运行对象。它并不提供您无法通过上述自定义函数实现的功能，因此我们建议使用自定义函数。

`RunnableBranch` 通过一组 (条件, 可运行对象) 对和一个默认可运行对象进行初始化。它通过将每个条件与调用时传入的输入进行比较来选择分支。它选择第一个评估为 True 的条件，并使用输入运行与该条件对应的可运行对象。

如果没有提供的条件匹配，它将运行默认可运行对象。

以下是其实际应用示例：

```python
from langchain_core.runnables import RunnableBranch

branch = RunnableBranch(
    (lambda x: "anthropic" in x["topic"].lower(), anthropic_chain),
    (lambda x: "langchain" in x["topic"].lower(), langchain_chain),
    general_chain,
)
full_chain = {"topic": chain, "question": lambda x: x["question"]} | branch
full_chain.invoke({"question": "how do I use Anthropic?"})
```



```output
AIMessage(content="As Dario Amodei told me, to use Anthropic, you should first familiarize yourself with our mission and principles. Anthropic is committed to developing safe and beneficial artificial intelligence that can help solve important problems facing humanity. \n\nTo get started, I recommend exploring the resources on our website, which cover our research, products, and approach to AI development. You can also reach out to our team to learn more about how Anthropic's technology and services can support your specific needs.\n\nThe key is to engage with us in a way that aligns with our values of transparency, ethical AI, and a commitment to the wellbeing of humanity. We're here to collaborate and help you harness the power of advanced AI responsibly.", response_metadata={'id': 'msg_0187BVnpniPDJnVvwf3M1LdY', 'content': [ContentBlock(text="As Dario Amodei told me, to use Anthropic, you should first familiarize yourself with our mission and principles. Anthropic is committed to developing safe and beneficial artificial intelligence that can help solve important problems facing humanity. \n\nTo get started, I recommend exploring the resources on our website, which cover our research, products, and approach to AI development. You can also reach out to our team to learn more about how Anthropic's technology and services can support your specific needs.\n\nThe key is to engage with us in a way that aligns with our values of transparency, ethical AI, and a commitment to the wellbeing of humanity. We're here to collaborate and help you harness the power of advanced AI responsibly.", type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=53, output_tokens=160)})
```



```python
full_chain.invoke({"question": "how do I use LangChain?"})
```



```output
AIMessage(content="As Harrison Chase told me, using LangChain involves several key steps. First, you'll need to install the LangChain library and import the necessary modules. Then, you'll want to define your language model, any data sources you plan to use, and the specific tasks you want to accomplish, such as question answering, text generation, or agent-based reasoning. \n\nLangChain provides a flexible framework for building applications that leverage large language models. It includes abstractions for things like retrievers, prompts, and chains, which allow you to compose different components together to create powerful workflows. \n\nThe documentation on the LangChain website is excellent and covers many common use cases in detail. I'd recommend starting there to get a solid understanding of the core concepts and how to apply them to your specific needs. And of course, feel free to reach out if you have any other questions - I'm always happy to share more insights from my conversations with Harrison.", response_metadata={'id': 'msg_01T1naS99wGPkEAP4LME8iAv', 'content': [ContentBlock(text="As Harrison Chase told me, using LangChain involves several key steps. First, you'll need to install the LangChain library and import the necessary modules. Then, you'll want to define your language model, any data sources you plan to use, and the specific tasks you want to accomplish, such as question answering, text generation, or agent-based reasoning. \n\nLangChain provides a flexible framework for building applications that leverage large language models. It includes abstractions for things like retrievers, prompts, and chains, which allow you to compose different components together to create powerful workflows. \n\nThe documentation on the LangChain website is excellent and covers many common use cases in detail. I'd recommend starting there to get a solid understanding of the core concepts and how to apply them to your specific needs. And of course, feel free to reach out if you have any other questions - I'm always happy to share more insights from my conversations with Harrison.", type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=50, output_tokens=205)})
```



```python
full_chain.invoke({"question": "whats 2 + 2"})
```



```output
AIMessage(content='4', response_metadata={'id': 'msg_01T6T3TS6hRCtU8JayN93QEi', 'content': [ContentBlock(text='4', type='text')], 'model': 'claude-3-haiku-20240307', 'role': 'assistant', 'stop_reason': 'end_turn', 'stop_sequence': None, 'type': 'message', 'usage': Usage(input_tokens=28, output_tokens=5)})
```

## 语义相似度路由

一种特别有用的技术是使用嵌入将查询路由到最相关的提示。以下是一个示例。

```python
from langchain_community.utils.math import cosine_similarity
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import PromptTemplate
from langchain_core.runnables import RunnableLambda, RunnablePassthrough
from langchain_openai import OpenAIEmbeddings

physics_template = """You are a very smart physics professor. \
You are great at answering questions about physics in a concise and easy to understand manner. \
When you don't know the answer to a question you admit that you don't know.

Here is a question:
{query}"""

math_template = """You are a very good mathematician. You are great at answering math questions. \
You are so good because you are able to break down hard problems into their component parts, \
answer the component parts, and then put them together to answer the broader question.

Here is a question:
{query}"""

embeddings = OpenAIEmbeddings()
prompt_templates = [physics_template, math_template]
prompt_embeddings = embeddings.embed_documents(prompt_templates)


def prompt_router(input):
    query_embedding = embeddings.embed_query(input["query"])
    similarity = cosine_similarity([query_embedding], prompt_embeddings)[0]
    most_similar = prompt_templates[similarity.argmax()]
    print("使用数学" if most_similar == math_template else "使用物理")
    return PromptTemplate.from_template(most_similar)


chain = (
    {"query": RunnablePassthrough()}
    | RunnableLambda(prompt_router)
    | ChatAnthropic(model="claude-3-haiku-20240307")
    | StrOutputParser()
)
```


```python
print(chain.invoke("What's a black hole"))
```
```output
使用物理
作为一名物理教授，我很高兴能提供关于黑洞的简明易懂的解释。

黑洞是一个极其密集的时空区域，其引力如此强大，以至于没有任何东西，甚至光，都无法逃脱。这意味着如果你靠近黑洞，你会被强大的引力所吸引并压碎。

黑洞的形成发生在一个比我们的太阳大得多的巨大恒星达到生命终点并坍缩自身时。这种坍缩使物质变得极其密集，引力变得如此强大，以至于形成了一个不可逆转的点，称为事件视界。

在事件视界之外，我们所知道的物理定律失效，强大的引力产生了一个奇点，这是时空中无限密度和曲率的点。

黑洞是迷人而神秘的物体，关于它们的性质和行为仍有许多需要了解的地方。如果我对黑洞的任何具体细节或方面不确定，我会坦诚地承认我并没有完全理解，并鼓励进一步的研究和调查。
```

```python
print(chain.invoke("What's a path integral"))
```
```output
使用数学
路径积分是物理学中一个强大的数学概念，特别是在量子力学领域。它是由著名物理学家理查德·费曼发展起来的，作为量子力学的一种替代表述。

在路径积分中，不再考虑粒子从一个点到另一个点的单一路径，正如经典力学中那样，而是考虑粒子同时采取所有可能的路径。每条路径都被赋予一个复值权重，粒子从一个点到另一个点的总概率振幅通过对所有可能路径进行求和（积分）来计算。

路径积分表述背后的关键思想有：

1. 叠加原理：在量子力学中，粒子可以同时存在于多种状态或路径的叠加中。

2. 概率振幅：粒子从一个点到另一个点的概率振幅是通过对所有可能路径的复值权重进行求和来计算的。

3. 路径加权：每条路径根据沿该路径的作用（拉格朗日量的时间积分）赋予权重。作用较低的路径具有更大的权重。

4. 费曼的方法：费曼将路径积分表述发展为量子力学中传统波函数方法的替代方案，提供了对量子现象更直观和概念性的理解。

路径积分方法在量子场论中特别有用，它为计算转变概率和理解量子系统的行为提供了强大的框架。它还在物理学的各个领域找到了应用，如凝聚态、统计力学，甚至在金融学中（路径积分在期权定价中的应用）。

路径积分的数学构造涉及功能分析和测度理论中的高级概念，使其成为物理学家工具箱中的强大而复杂的工具。
```

## 下一步

您现在已经学习了如何为您的组合 LCEL 链添加路由。

接下来，查看本节中关于可运行项的其他操作指南。