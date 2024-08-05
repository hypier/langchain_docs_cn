---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/tutorials/llm_chain.ipynb
sidebar_position: 0
---

# 使用 LCEL 构建简单的 LLM 应用程序

在这个快速入门中，我们将向您展示如何使用 LangChain 构建一个简单的 LLM 应用程序。该应用程序将把文本从英语翻译成另一种语言。这是一个相对简单的 LLM 应用程序 - 它只是一次 LLM 调用加上一些提示。尽管如此，这仍然是开始使用 LangChain 的好方法 - 只需一些提示和一次 LLM 调用，就可以构建许多功能！

在阅读完本教程后，您将对以下内容有一个高层次的了解：

- 使用 [language models](/docs/concepts/#chat-models)

- 使用 [PromptTemplates](/docs/concepts/#prompt-templates) 和 [OutputParsers](/docs/concepts/#output-parsers)

- 使用 [LangChain Expression Language (LCEL)](/docs/concepts/#langchain-expression-language-lcel) 将组件串联在一起

- 使用 [LangSmith](/docs/concepts/#langsmith) 调试和跟踪您的应用程序

- 使用 [LangServe](/docs/concepts/#langserve) 部署您的应用程序

让我们开始吧！

## 设置

### Jupyter Notebook

本指南（以及文档中的大多数其他指南）使用 [Jupyter notebooks](https://jupyter.org/) 并假设读者也是如此。Jupyter notebooks 非常适合学习如何使用 LLM 系统，因为有时事情可能会出错（意外输出、API 停止工作等），在交互式环境中逐步阅读指南是更好地理解它们的好方法。

本教程和其他教程最方便的运行方式可能是在 Jupyter notebook 中。有关如何安装的说明，请参见 [这里](https://jupyter.org/install)。

### 安装

要安装 LangChain，请运行：

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CodeBlock from "@theme/CodeBlock";

<Tabs>
  <TabItem value="pip" label="Pip" default>
    <CodeBlock language="bash">pip install langchain</CodeBlock>
  </TabItem>
  <TabItem value="conda" label="Conda">
    <CodeBlock language="bash">conda install langchain -c conda-forge</CodeBlock>
  </TabItem>
</Tabs>

有关更多详细信息，请参阅我们的 [安装指南](/docs/how_to/installation)。

### LangSmith

您使用 LangChain 构建的许多应用程序将包含多个步骤和多次调用 LLM。  
随着这些应用程序变得越来越复杂，能够检查您的链或代理内部到底发生了什么变得至关重要。  
做到这一点的最佳方法是使用 [LangSmith](https://smith.langchain.com)。

在您在上述链接注册后，请确保设置您的环境变量以开始记录跟踪：

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

## 使用语言模型

首先，让我们学习如何单独使用语言模型。LangChain 支持许多不同的语言模型，您可以互换使用 - 请选择您想要使用的模型！

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs openaiParams={`model="gpt-4"`} />

我们首先直接使用该模型。`ChatModel` 是 LangChain "Runnables" 的实例，这意味着它们暴露了一个标准接口以便与之交互。为了简单地调用模型，我们可以将一系列消息传递给 `.invoke` 方法。

```python
from langchain_core.messages import HumanMessage, SystemMessage

messages = [
    SystemMessage(content="Translate the following from English into Italian"),
    HumanMessage(content="hi!"),
]

model.invoke(messages)
```

```output
AIMessage(content='ciao!', response_metadata={'token_usage': {'completion_tokens': 3, 'prompt_tokens': 20, 'total_tokens': 23}, 'model_name': 'gpt-4', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-fc5d7c88-9615-48ab-a3c7-425232b562c5-0')
```

如果我们启用了 LangSmith，我们可以看到此运行被记录到 LangSmith，并可以查看 [LangSmith 跟踪](https://smith.langchain.com/public/88baa0b2-7c1a-4d09-ba30-a47985dde2ea/r)

## OutputParsers

请注意，模型的响应是一个 `AIMessage`。这包含一个字符串响应以及关于响应的其他元数据。我们通常只想处理字符串响应。我们可以使用一个简单的输出解析器来解析出这个响应。

我们首先导入简单的输出解析器。

```python
from langchain_core.output_parsers import StrOutputParser

parser = StrOutputParser()
```

使用它的一种方式是单独使用它。例如，我们可以保存语言模型调用的结果，然后将其传递给解析器。

```python
result = model.invoke(messages)
```

```python
parser.invoke(result)
```

```output
'Ciao!'
```

更常见的是，我们可以将模型与这个输出解析器“链式”连接。这意味着在这个链中，每次都会调用这个输出解析器。这个链接受语言模型的输入类型（字符串或消息列表）并返回输出解析器的输出类型（字符串）。

我们可以轻松地使用 `|` 运算符创建这个链。`|` 运算符在 LangChain 中用于将两个元素组合在一起。

```python
chain = model | parser
```

```python
chain.invoke(messages)
```

```output
'Ciao!'
```

如果我们现在查看 LangSmith，我们可以看到这个链有两个步骤：首先调用语言模型，然后将结果传递给输出解析器。我们可以看到 [LangSmith trace]( https://smith.langchain.com/public/f1bdf656-2739-42f7-ac7f-0f1dd712322f/r)

## 提示模板

现在我们直接将一系列消息传递给语言模型。这些消息列表来自哪里？通常，它是由用户输入和应用逻辑的组合构建而成的。这个应用逻辑通常会将原始用户输入转换为准备传递给语言模型的消息列表。常见的转换包括添加系统消息或使用用户输入格式化模板。

PromptTemplates 是 LangChain 中的一个概念，旨在帮助进行这种转换。它们接受原始用户输入，并返回准备传递给语言模型的数据（提示）。

让我们在这里创建一个 PromptTemplate。它将接受两个用户变量：

- `language`: 要翻译成的语言
- `text`: 要翻译的文本


```python
from langchain_core.prompts import ChatPromptTemplate
```

首先，让我们创建一个字符串，将其格式化为系统消息：


```python
system_template = "Translate the following into {language}:"
```

接下来，我们可以创建 PromptTemplate。这将是 `system_template` 和一个更简单的模板的组合，用于放置要翻译的文本


```python
prompt_template = ChatPromptTemplate.from_messages(
    [("system", system_template), ("user", "{text}")]
)
```

这个提示模板的输入是一个字典。我们可以单独玩一下这个提示模板，看看它的作用


```python
result = prompt_template.invoke({"language": "italian", "text": "hi"})

result
```



```output
ChatPromptValue(messages=[SystemMessage(content='Translate the following into italian:'), HumanMessage(content='hi')])
```


我们可以看到它返回了一个 `ChatPromptValue`，包含两条消息。如果我们想直接访问这些消息，可以这样做：


```python
result.to_messages()
```



```output
[SystemMessage(content='Translate the following into italian:'),
 HumanMessage(content='hi')]
```

## 使用 LCEL 连接组件

我们现在可以使用管道 (`|`) 操作符将其与上述模型和输出解析器结合起来：


```python
chain = prompt_template | model | parser
```


```python
chain.invoke({"language": "italian", "text": "hi"})
```



```output
'ciao'
```


这是一个使用 [LangChain 表达式语言 (LCEL)](/docs/concepts/#langchain-expression-language-lcel) 将 LangChain 模块连接在一起的简单示例。这种方法有几个好处，包括优化的流处理和追踪支持。

如果我们查看 LangSmith 追踪，我们可以看到所有三个组件出现在 [LangSmith 追踪](https://smith.langchain.com/public/bc49bec0-6b13-4726-967f-dbd3448b786d/r) 中。

## 使用 LangServe 提供服务

现在我们已经构建了一个应用程序，我们需要提供服务。这就是 LangServe 的用武之地。
LangServe 帮助开发者将 LangChain 链部署为 REST API。您不需要使用 LangServe 来使用 LangChain，但在本指南中，我们将展示如何使用 LangServe 部署您的应用。

虽然本指南的第一部分旨在 Jupyter Notebook 或脚本中运行，但我们现在将移出该环境。我们将创建一个 Python 文件，然后从命令行与之交互。

安装命令：
```bash
pip install "langserve[all]"
```

### 服务器

为了为我们的应用程序创建一个服务器，我们将创建一个 `serve.py` 文件。该文件将包含我们应用程序的服务逻辑。它包括三部分：
1. 我们刚刚构建的链的定义
2. 我们的 FastAPI 应用
3. 从中提供链的路由定义，这通过 `langserve.add_routes` 来完成


```python
#!/usr/bin/env python
from typing import List

from fastapi import FastAPI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import ChatOpenAI
from langserve import add_routes

# 1. 创建提示模板
system_template = "Translate the following into {language}:"
prompt_template = ChatPromptTemplate.from_messages([
    ('system', system_template),
    ('user', '{text}')
])

# 2. 创建模型
model = ChatOpenAI()

# 3. 创建解析器
parser = StrOutputParser()

# 4. 创建链
chain = prompt_template | model | parser


# 4. 应用定义
app = FastAPI(
  title="LangChain 服务器",
  version="1.0",
  description="一个使用 LangChain 的 Runnable 接口的简单 API 服务器",
)

# 5. 添加链路由

add_routes(
    app,
    chain,
    path="/chain",
)

if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="localhost", port=8000)
```

就这样！如果我们执行这个文件：
```bash
python serve.py
```
我们应该在 [http://localhost:8000](http://localhost:8000) 看到我们的链被提供。

### 游乐场

每个 LangServe 服务都配备了一个简单的 [内置用户界面](https://github.com/langchain-ai/langserve/blob/main/README.md#playground)，用于配置和调用应用程序，具有流式输出和对中间步骤的可见性。  
前往 [http://localhost:8000/chain/playground/](http://localhost:8000/chain/playground/) 试试吧！输入与之前相同的内容 - `{"language": "italian", "text": "hi"}` - 它应该会与之前的响应相同。

### 客户端

现在让我们设置一个客户端，以便以编程方式与我们的服务进行交互。我们可以很容易地使用 [langserve.RemoteRunnable](/docs/langserve/#client) 来实现这一点。
使用这个，我们可以像在客户端运行一样与服务的链进行交互。


```python
from langserve import RemoteRunnable

remote_chain = RemoteRunnable("http://localhost:8000/chain/")
remote_chain.invoke({"language": "italian", "text": "hi"})
```



```output
'Ciao'
```


要了解更多关于 LangServe 的其他功能，请 [点击这里](/docs/langserve)。

## 结论

就这些！在本教程中，您已经学习了如何创建第一个简单的 LLM 应用程序。您了解了如何使用语言模型，如何解析它们的输出，如何创建提示模板，如何将它们与 LCEL 链接，如何通过 LangSmith 获取对您创建的链的出色可观察性，以及如何使用 LangServe 部署它们。

这只是您希望学习以成为熟练的 AI 工程师的表面。幸运的是，我们还有很多其他资源！

有关 LangChain 核心概念的进一步阅读，我们提供了详细的 [概念指南](/docs/concepts)。

如果您对这些概念有更具体的问题，请查看以下如何指南的部分：

- [LangChain 表达语言 (LCEL)](/docs/how_to/#langchain-expression-language-lcel)
- [提示模板](/docs/how_to/#prompt-templates)
- [聊天模型](/docs/how_to/#chat-models)
- [输出解析器](/docs/how_to/#output-parsers)
- [LangServe](/docs/langserve/)

还有 LangSmith 文档：

- [LangSmith](https://docs.smith.langchain.com)