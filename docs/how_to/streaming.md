---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/streaming.ipynb
keywords: [流]
---

# 如何流式运行可执行对象

:::info 前提条件

本指南假设您熟悉以下概念：
- [聊天模型](/docs/concepts/#chat-models)
- [LangChain 表达语言](/docs/concepts/#langchain-expression-language)
- [输出解析器](/docs/concepts/#output-parsers)

:::

流式处理对于基于 LLM 的应用程序让终端用户感觉响应迅速至关重要。

重要的 LangChain 原语，如 [聊天模型](/docs/concepts/#chat-models)、[输出解析器](/docs/concepts/#output-parsers)、[提示](/docs/concepts/#prompt-templates)、[检索器](/docs/concepts/#retrievers) 和 [代理](/docs/concepts/#agents) 实现了 LangChain [可执行接口](/docs/concepts#interface)。

该接口提供了两种流式内容的通用方法：

1. 同步 `stream` 和异步 `astream`：一种 **默认实现** 的流式处理，从链中流式传输 **最终输出**。
2. 异步 `astream_events` 和异步 `astream_log`：这些提供了一种从链中流式传输 **中间步骤** 和 **最终输出** 的方法。

让我们看看这两种方法，并尝试理解如何使用它们。

:::info
有关 LangChain 中流式技术的更高层次概述，请参见 [概念指南的这一部分](/docs/concepts/#streaming)。
:::

## 使用流

所有 `Runnable` 对象都实现了一个名为 `stream` 的同步方法和一个名为 `astream` 的异步变体。

这些方法旨在以块的形式流式传输最终输出，尽快返回每个块。

只有当程序中的所有步骤都知道如何处理 **输入流** 时，才能进行流式传输；即，一次处理一个输入块，并产生相应的输出块。

这种处理的复杂性可以有所不同，从简单的任务，如发出 LLM 生成的标记，到更具挑战性的任务，如在整个 JSON 完成之前流式传输 JSON 结果的部分。

探索流式传输的最佳起点是 LLM 应用程序中最重要的组件——LLM 本身！

### LLMs 和聊天模型

大型语言模型及其聊天变体是基于 LLM 应用的主要瓶颈。

大型语言模型生成完整响应查询可能需要 **几秒钟**。这远慢于应用程序对最终用户感觉响应的 **~200-300 毫秒** 阈值。

使应用程序感觉更具响应性的关键策略是展示中间进展；即，逐个 **token** 流式输出模型的结果。

我们将展示使用聊天模型的流式示例。请从以下选项中选择一个：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs
  customVarName="model"
/>

让我们从同步的 `stream` API 开始：


```python
chunks = []
for chunk in model.stream("what color is the sky?"):
    chunks.append(chunk)
    print(chunk.content, end="|", flush=True)
```
```output
The| sky| appears| blue| during| the| day|.|
```
或者，如果您在异步环境中工作，您可以考虑使用异步的 `astream` API：


```python
chunks = []
async for chunk in model.astream("what color is the sky?"):
    chunks.append(chunk)
    print(chunk.content, end="|", flush=True)
```
```output
The| sky| appears| blue| during| the| day|.|
```
让我们检查其中一个块


```python
chunks[0]
```



```output
AIMessageChunk(content='The', id='run-b36bea64-5511-4d7a-b6a3-a07b3db0c8e7')
```


我们得到了一个叫做 `AIMessageChunk` 的东西。这个块代表了一个 `AIMessage` 的一部分。

消息块的设计是可叠加的——可以简单地将它们相加以获取到目前为止的响应状态！


```python
chunks[0] + chunks[1] + chunks[2] + chunks[3] + chunks[4]
```



```output
AIMessageChunk(content='The sky appears blue during', id='run-b36bea64-5511-4d7a-b6a3-a07b3db0c8e7')
```

### 链

几乎所有的 LLM 应用都涉及比仅仅调用语言模型更多的步骤。

让我们使用 `LangChain 表达式语言` (`LCEL`) 构建一个简单的链，该链结合了提示、模型和解析器，并验证流式输出是否有效。

我们将使用 [`StrOutputParser`](https://api.python.langchain.com/en/latest/output_parsers/langchain_core.output_parsers.string.StrOutputParser.html) 来解析模型的输出。这是一个简单的解析器，它从 `AIMessageChunk` 中提取 `content` 字段，从而获取模型返回的 `token`。

:::tip
LCEL 是一种 *声明性* 的方式，通过将不同的 LangChain 原语串联在一起指定一个“程序”。使用 LCEL 创建的链受益于 `stream` 和 `astream` 的自动实现，允许最终输出的流式传输。实际上，使用 LCEL 创建的链实现了整个标准的 Runnable 接口。
:::


```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_template("tell me a joke about {topic}")
parser = StrOutputParser()
chain = prompt | model | parser

async for chunk in chain.astream({"topic": "parrot"}):
    print(chunk, end="|", flush=True)
```
```output
Here|'s| a| joke| about| a| par|rot|:|

A man| goes| to| a| pet| shop| to| buy| a| par|rot|.| The| shop| owner| shows| him| two| stunning| pa|rr|ots| with| beautiful| pl|um|age|.|

"|There|'s| a| talking| par|rot| an|d a| non|-|talking| par|rot|,"| the| owner| says|.| "|The| talking| par|rot| costs| $|100|,| an|d the| non|-|talking| par|rot| is| $|20|."|

The| man| says|,| "|I|'ll| take| the| non|-|talking| par|rot| at| $|20|."|

He| pays| an|d leaves| with| the| par|rot|.| As| he|'s| walking| down| the| street|,| the| par|rot| looks| up| at| him| an|d says|,| "|You| know|,| you| really| are| a| stupi|d man|!"|

The| man| is| stun|ne|d an|d looks| at| the| par|rot| in| dis|bel|ief|.| The| par|rot| continues|,| "|Yes|,| you| got| r|ippe|d off| big| time|!| I| can| talk| just| as| well| as| that| other| par|rot|,| an|d you| only| pai|d $|20| |for| me|!"|
```
注意，即使在上面的链中使用了 `parser`，我们仍然得到了流式输出。`parser` 针对每个流式块单独操作。许多 [LCEL 原语](/docs/how_to#langchain-expression-language-lcel) 也支持这种变换风格的直通流式传输，这在构建应用时非常方便。

自定义函数可以被 [设计为返回生成器](/docs/how_to/functions#streaming)，这些生成器能够在流上操作。

某些可运行对象，如 [提示模板](/docs/how_to#prompt-templates) 和 [聊天模型](/docs/how_to#chat-models)，无法处理单个块，而是聚合所有先前的步骤。这类可运行对象可能会中断流式处理。

:::note
LangChain 表达式语言允许您将链的构建与其使用模式（例如，同步/异步、批处理/流式等）分开。如果这与您构建的内容无关，您还可以依赖标准的 **命令式** 编程方法，通过分别调用 `invoke`、`batch` 或 `stream` 在每个组件上，分配结果给变量，然后根据需要在下游使用它们。
:::

### 处理输入流

如果您想要在生成输出时流式传输 JSON，会怎么样呢？

如果您依赖于 `json.loads` 来解析部分 JSON，解析将失败，因为部分 JSON 不是有效的 JSON。

您可能会完全不知道该怎么办，并声称无法流式传输 JSON。

实际上，有一种方法可以做到这一点——解析器需要在 **输入流** 上操作，并尝试将部分 JSON “自动完成” 到有效状态。

让我们看看这样的解析器是如何工作的，以理解这意味着什么。


```python
from langchain_core.output_parsers import JsonOutputParser

chain = (
    model | JsonOutputParser()
)  # 由于 Langchain 较旧版本中的一个错误，JsonOutputParser 并未从某些模型中流式传输结果
async for text in chain.astream(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`"
):
    print(text, flush=True)
```
```output
{}
{'countries': []}
{'countries': [{}]}
{'countries': [{'name': ''}]}
{'countries': [{'name': 'France'}]}
{'countries': [{'name': 'France', 'population': 67}]}
{'countries': [{'name': 'France', 'population': 67413}]}
{'countries': [{'name': 'France', 'population': 67413000}]}
{'countries': [{'name': 'France', 'population': 67413000}, {}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': ''}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain'}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {'name': ''}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {'name': 'Japan'}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {'name': 'Japan', 'population': 125}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {'name': 'Japan', 'population': 125584}]}
{'countries': [{'name': 'France', 'population': 67413000}, {'name': 'Spain', 'population': 47351567}, {'name': 'Japan', 'population': 125584000}]}
```
现在，让我们**破坏**流式传输。我们将使用前面的示例，并在末尾附加一个提取函数，该函数从最终 JSON 中提取国家名称。

:::warning
链中任何处理 **最终输入** 而不是 **输入流** 的步骤都可能通过 `stream` 或 `astream` 破坏流式传输功能。
:::

:::tip
稍后，我们将讨论 `astream_events` API，该 API 从中间步骤流式传输结果。即使链中包含仅对 **最终输入** 操作的步骤，该 API 也会从中间步骤流式传输结果。
:::


```python
from langchain_core.output_parsers import (
    JsonOutputParser,
)


# 一个处理最终输入而不是输入流的函数
def _extract_country_names(inputs):
    """一个不处理输入流并破坏流式传输的函数。"""
    if not isinstance(inputs, dict):
        return ""

    if "countries" not in inputs:
        return ""

    countries = inputs["countries"]

    if not isinstance(countries, list):
        return ""

    country_names = [
        country.get("name") for country in countries if isinstance(country, dict)
    ]
    return country_names


chain = model | JsonOutputParser() | _extract_country_names

async for text in chain.astream(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`"
):
    print(text, end="|", flush=True)
```
```output
['France', 'Spain', 'Japan']|
```
#### 生成器函数

让我们使用一个可以在 **输入流** 上操作的生成器函数来修复流式传输。

:::tip
生成器函数（使用 `yield` 的函数）允许编写在 **输入流** 上操作的代码。
:::


```python
from langchain_core.output_parsers import JsonOutputParser


async def _extract_country_names_streaming(input_stream):
    """一个在输入流上操作的函数。"""
    country_names_so_far = set()

    async for input in input_stream:
        if not isinstance(input, dict):
            continue

        if "countries" not in input:
            continue

        countries = input["countries"]

        if not isinstance(countries, list):
            continue

        for country in countries:
            name = country.get("name")
            if not name:
                continue
            if name not in country_names_so_far:
                yield name
                country_names_so_far.add(name)


chain = model | JsonOutputParser() | _extract_country_names_streaming

async for text in chain.astream(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`",
):
    print(text, end="|", flush=True)
```
```output
France|Spain|Japan|
```
:::note
由于上面的代码依赖于 JSON 自动完成，您可能会看到国家的部分名称（例如，`Sp` 和 `Spain`），这不是提取结果所希望的！

我们关注的是流式传输的概念，而不一定是链的结果。
:::

### 非流式组件

一些内置组件如检索器不提供任何 `streaming`。如果我们尝试对它们进行 `stream` 会发生什么呢？🤨


```python
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import OpenAIEmbeddings

template = """Answer the question based only on the following context:
{context}

Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)

vectorstore = FAISS.from_texts(
    ["harrison worked at kensho", "harrison likes spicy food"],
    embedding=OpenAIEmbeddings(),
)
retriever = vectorstore.as_retriever()

chunks = [chunk for chunk in retriever.stream("where did harrison work?")]
chunks
```



```output
[[Document(page_content='harrison worked at kensho'),
  Document(page_content='harrison likes spicy food')]]
```


Stream 仅返回该组件的最终结果。

这没问题 🥹！并不是所有组件都必须实现流式处理——在某些情况下，流式处理要么不必要，要么困难，或者根本没有意义。

:::tip
使用非流式组件构建的 LCEL 链，在很多情况下仍然能够进行流式处理，流式输出的部分将在链中最后一个非流式步骤之后开始。
:::


```python
retrieval_chain = (
    {
        "context": retriever.with_config(run_name="Docs"),
        "question": RunnablePassthrough(),
    }
    | prompt
    | model
    | StrOutputParser()
)
```


```python
for chunk in retrieval_chain.stream(
    "Where did harrison work? " "Write 3 made up sentences about this place."
):
    print(chunk, end="|", flush=True)
```
```output
Base|d on| the| given| context|,| Harrison| worke|d at| K|ens|ho|.|

Here| are| |3| |made| up| sentences| about| this| place|:|

1|.| K|ens|ho| was| a| cutting|-|edge| technology| company| known| for| its| innovative| solutions| in| artificial| intelligence| an|d data| analytics|.|

2|.| The| modern| office| space| at| K|ens|ho| feature|d open| floor| plans|,| collaborative| work|sp|aces|,| an|d a| vib|rant| atmosphere| that| fos|tere|d creativity| an|d team|work|.|

3|.| With| its| prime| location| in| the| heart| of| the| city|,| K|ens|ho| attracte|d top| talent| from| aroun|d the| worl|d,| creating| a| diverse| an|d dynamic| work| environment|.|
```
现在我们已经了解了 `stream` 和 `astream` 的工作原理，让我们进入流式事件的世界。🏞️

## 使用流事件

事件流是一个**beta** API。根据反馈，此API可能会有所更改。

:::note

本指南演示了`V2` API，并要求 langchain-core >= 0.2。有关与旧版本 LangChain 兼容的 `V1` API，请参见 [这里](https://python.langchain.com/v0.1/docs/expression_language/streaming/#using-stream-events)。
:::


```python
import langchain_core

langchain_core.__version__
```

为了使 `astream_events` API 正常工作：

* 尽可能在代码中使用 `async`（例如，异步工具等）
* 如果定义自定义函数/可运行对象，请传播回调
* 每当使用没有 LCEL 的可运行对象时，请确保在 LLM 上调用 `.astream()` 而不是 `.ainvoke` 以强制 LLM 流式传输令牌。
* 如果有任何问题，请告诉我们！ :)

### 事件参考

以下是一个参考表，显示了各种可运行对象可能发出的事件。

:::note
当流处理正确实现时，Runnable 的输入在输入流完全消耗之前是未知的。这意味着 `inputs` 通常只会在 `end` 事件中包含，而不是在 `start` 事件中。
:::

| 事件                  | 名称              | 块                               | 输入                                         | 输出                                          |
|----------------------|------------------|---------------------------------|-----------------------------------------------|-------------------------------------------------|
| on_chat_model_start  | [model name]     |                                 | {"messages": [[SystemMessage, HumanMessage]]} |                                                 |
| on_chat_model_stream | [model name]     | AIMessageChunk(content="hello") |                                               |                                                 |
| on_chat_model_end    | [model name]     |                                 | {"messages": [[SystemMessage, HumanMessage]]} | AIMessageChunk(content="hello world")           |
| on_llm_start         | [model name]     |                                 | {'input': 'hello'}                            |                                                 |
| on_llm_stream        | [model name]     | 'Hello'                         |                                               |                                                 |
| on_llm_end           | [model name]     |                                 | 'Hello human!'                                |                                                 |
| on_chain_start       | format_docs      |                                 |                                               |                                                 |
| on_chain_stream      | format_docs      | "hello world!, goodbye world!"  |                                               |                                                 |
| on_chain_end         | format_docs      |                                 | [Document(...)]                               | "hello world!, goodbye world!"                  |
| on_tool_start        | some_tool        |                                 | {"x": 1, "y": "2"}                            |                                                 |
| on_tool_end          | some_tool        |                                 |                                               | {"x": 1, "y": "2"}                              |
| on_retriever_start   | [retriever name] |                                 | {"query": "hello"}                            |                                                 |
| on_retriever_end     | [retriever name] |                                 | {"query": "hello"}                            | [Document(...), ..]                             |
| on_prompt_start      | [template_name]  |                                 | {"question": "hello"}                         |                                                 |
| on_prompt_end        | [template_name]  |                                 | {"question": "hello"}                         | ChatPromptValue(messages: [SystemMessage, ...]) |

### 聊天模型

让我们先来看看聊天模型生成的事件。


```python
events = []
async for event in model.astream_events("hello", version="v2"):
    events.append(event)
```
```output
/home/eugene/src/langchain/libs/core/langchain_core/_api/beta_decorator.py:87: LangChainBetaWarning: This API is in beta and may change in the future.
  warn_beta(
```
:::note

嘿，API 中那个有趣的参数 version="v2" 是什么？！ 😾

这是一个 **beta API**，我们几乎肯定会对其进行一些更改（实际上，我们已经做过了！）

这个版本参数将使我们能够将此类破坏性更改对您的代码的影响降到最低。

简而言之，我们现在让您烦恼，以便以后不再让您烦恼。

`v2` 仅适用于 langchain-core>=0.2.0。

:::

让我们来看一下几个开始事件和几个结束事件。


```python
events[:3]
```



```output
[{'event': 'on_chat_model_start',
  'data': {'input': 'hello'},
  'name': 'ChatAnthropic',
  'tags': [],
  'run_id': 'a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3',
  'metadata': {}},
 {'event': 'on_chat_model_stream',
  'data': {'chunk': AIMessageChunk(content='Hello', id='run-a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3')},
  'run_id': 'a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3',
  'name': 'ChatAnthropic',
  'tags': [],
  'metadata': {}},
 {'event': 'on_chat_model_stream',
  'data': {'chunk': AIMessageChunk(content='!', id='run-a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3')},
  'run_id': 'a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3',
  'name': 'ChatAnthropic',
  'tags': [],
  'metadata': {}}]
```



```python
events[-2:]
```



```output
[{'event': 'on_chat_model_stream',
  'data': {'chunk': AIMessageChunk(content='?', id='run-a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3')},
  'run_id': 'a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3',
  'name': 'ChatAnthropic',
  'tags': [],
  'metadata': {}},
 {'event': 'on_chat_model_end',
  'data': {'output': AIMessageChunk(content='Hello! How can I assist you today?', id='run-a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3')},
  'run_id': 'a81e4c0f-fc36-4d33-93bc-1ac25b9bb2c3',
  'name': 'ChatAnthropic',
  'tags': [],
  'metadata': {}}]
```

### Chain

让我们回顾一下解析流式 JSON 的示例链，以探索流式事件 API。

```python
chain = (
    model | JsonOutputParser()
)  # 由于 Langchain 的早期版本中的一个错误，JsonOutputParser 没有从某些模型流式输出结果

events = [
    event
    async for event in chain.astream_events(
        "output a list of the countries france, spain and japan and their populations in JSON format. "
        'Use a dict with an outer key of "countries" which contains a list of countries. '
        "Each country should have the key `name` and `population`",
        version="v2",
    )
]
```

如果你检查前几个事件，你会注意到有 **3** 个不同的开始事件，而不是 **2** 个开始事件。

这三个开始事件对应于：

1. 链（模型 + 解析器）
2. 模型
3. 解析器

```python
events[:3]
```

```output
[{'event': 'on_chain_start',
  'data': {'input': 'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`'},
  'name': 'RunnableSequence',
  'tags': [],
  'run_id': '4765006b-16e2-4b1d-a523-edd9fd64cb92',
  'metadata': {}},
 {'event': 'on_chat_model_start',
  'data': {'input': {'messages': [[HumanMessage(content='output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`')]]}},
  'name': 'ChatAnthropic',
  'tags': ['seq:step:1'],
  'run_id': '0320c234-7b52-4a14-ae4e-5f100949e589',
  'metadata': {}},
 {'event': 'on_chat_model_stream',
  'data': {'chunk': AIMessageChunk(content='{', id='run-0320c234-7b52-4a14-ae4e-5f100949e589')},
  'run_id': '0320c234-7b52-4a14-ae4e-5f100949e589',
  'name': 'ChatAnthropic',
  'tags': ['seq:step:1'],
  'metadata': {}}]
```

如果你查看最后 3 个事件，你认为会看到什么？中间的呢？

让我们使用这个 API 输出模型和解析器的流式事件。我们忽略开始事件、结束事件和链中的事件。

```python
num_events = 0

async for event in chain.astream_events(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`",
    version="v2",
):
    kind = event["event"]
    if kind == "on_chat_model_stream":
        print(
            f"Chat model chunk: {repr(event['data']['chunk'].content)}",
            flush=True,
        )
    if kind == "on_parser_stream":
        print(f"Parser chunk: {event['data']['chunk']}", flush=True)
    num_events += 1
    if num_events > 30:
        # 截断输出
        print("...")
        break
```
```output
Chat model chunk: '{'
Parser chunk: {}
Chat model chunk: '\n  '
Chat model chunk: '"'
Chat model chunk: 'countries'
Chat model chunk: '":'
Chat model chunk: ' ['
Parser chunk: {'countries': []}
Chat model chunk: '\n    '
Chat model chunk: '{'
Parser chunk: {'countries': [{}]}
Chat model chunk: '\n      '
Chat model chunk: '"'
Chat model chunk: 'name'
Chat model chunk: '":'
Chat model chunk: ' "'
Parser chunk: {'countries': [{'name': ''}]}
Chat model chunk: 'France'
Parser chunk: {'countries': [{'name': 'France'}]}
Chat model chunk: '",'
Chat model chunk: '\n      '
Chat model chunk: '"'
Chat model chunk: 'population'
...
```
因为模型和解析器都支持流式输出，所以我们实时看到来自两个组件的流式事件！这不是很酷吗？🦜

### 过滤事件

由于这个 API 生成了很多事件，因此能够对事件进行过滤是非常有用的。

您可以通过组件的 `name`、组件的 `tags` 或组件的 `type` 进行过滤。

#### 按名称过滤


```python
chain = model.with_config({"run_name": "model"}) | JsonOutputParser().with_config(
    {"run_name": "my_parser"}
)

max_events = 0
async for event in chain.astream_events(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`",
    version="v2",
    include_names=["my_parser"],
):
    print(event)
    max_events += 1
    if max_events > 10:
        # Truncate output
        print("...")
        break
```
```output
{'event': 'on_parser_start', 'data': {'input': 'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`'}, 'name': 'my_parser', 'tags': ['seq:step:2'], 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': []}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': ''}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France'}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France', 'population': 67}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France', 'population': 67413}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France', 'population': 67413000}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France', 'population': 67413000}, {}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {'countries': [{'name': 'France', 'population': 67413000}, {'name': ''}]}}, 'run_id': 'e058d750-f2c2-40f6-aa61-10f84cd671a9', 'name': 'my_parser', 'tags': ['seq:step:2'], 'metadata': {}}
...
```
#### 按类型过滤


```python
chain = model.with_config({"run_name": "model"}) | JsonOutputParser().with_config(
    {"run_name": "my_parser"}
)

max_events = 0
async for event in chain.astream_events(
    'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`',
    version="v2",
    include_types=["chat_model"],
):
    print(event)
    max_events += 1
    if max_events > 10:
        # Truncate output
        print("...")
        break
```
```output
{'event': 'on_chat_model_start', 'data': {'input': 'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`'}, 'name': 'model', 'tags': ['seq:step:1'], 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='{', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='\n  ', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='"', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='countries', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='":', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content=' [', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='\n    ', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='{', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='\n      ', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='"', id='run-db246792-2a91-4eb3-a14b-29658947065d')}, 'run_id': 'db246792-2a91-4eb3-a14b-29658947065d', 'name': 'model', 'tags': ['seq:step:1'], 'metadata': {}}
...
```
#### 按标签过滤

:::caution

标签由给定可运行组件的子组件继承。

如果您使用标签进行过滤，请确保这是您想要的。
:::


```python
chain = (model | JsonOutputParser()).with_config({"tags": ["my_chain"]})

max_events = 0
async for event in chain.astream_events(
    'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`',
    version="v2",
    include_tags=["my_chain"],
):
    print(event)
    max_events += 1
    if max_events > 10:
        # Truncate output
        print("...")
        break
```
```output
{'event': 'on_chain_start', 'data': {'input': 'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`'}, 'name': 'RunnableSequence', 'tags': ['my_chain'], 'run_id': 'fd68dd64-7a4d-4bdb-a0c2-ee592db0d024', 'metadata': {}}
{'event': 'on_chat_model_start', 'data': {'input': {'messages': [[HumanMessage(content='output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`')]]}}, 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='{', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
{'event': 'on_parser_start', 'data': {}, 'name': 'JsonOutputParser', 'tags': ['seq:step:2', 'my_chain'], 'run_id': 'afde30b9-beac-4b36-b4c7-dbbe423ddcdb', 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {}}, 'run_id': 'afde30b9-beac-4b36-b4c7-dbbe423ddcdb', 'name': 'JsonOutputParser', 'tags': ['seq:step:2', 'my_chain'], 'metadata': {}}
{'event': 'on_chain_stream', 'data': {'chunk': {}}, 'run_id': 'fd68dd64-7a4d-4bdb-a0c2-ee592db0d024', 'name': 'RunnableSequence', 'tags': ['my_chain'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='\n  ', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='"', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='countries', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='":', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content=' [', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
...
```

### 非流式组件

记得有些组件由于不操作**输入流**而不适合流式处理吗？

虽然这样的组件在使用`astream`时可能会中断最终输出的流式处理，但`astream_events`仍然会从支持流式处理的中间步骤中产生流式事件！


```python
# Function that does not support streaming.
# It operates on the finalizes inputs rather than
# operating on the input stream.
def _extract_country_names(inputs):
    """A function that does not operates on input streams and breaks streaming."""
    if not isinstance(inputs, dict):
        return ""

    if "countries" not in inputs:
        return ""

    countries = inputs["countries"]

    if not isinstance(countries, list):
        return ""

    country_names = [
        country.get("name") for country in countries if isinstance(country, dict)
    ]
    return country_names


chain = (
    model | JsonOutputParser() | _extract_country_names
)  # This parser only works with OpenAI right now
```

正如预期的那样，`astream` API无法正常工作，因为`_extract_country_names`不在流上操作。


```python
async for chunk in chain.astream(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`",
):
    print(chunk, flush=True)
```
```output
['France', 'Spain', 'Japan']
```
现在，让我们确认使用astream_events时，我们仍然能够从模型和解析器看到流式输出。


```python
num_events = 0

async for event in chain.astream_events(
    "output a list of the countries france, spain and japan and their populations in JSON format. "
    'Use a dict with an outer key of "countries" which contains a list of countries. '
    "Each country should have the key `name` and `population`",
    version="v2",
):
    kind = event["event"]
    if kind == "on_chat_model_stream":
        print(
            f"Chat model chunk: {repr(event['data']['chunk'].content)}",
            flush=True,
        )
    if kind == "on_parser_stream":
        print(f"Parser chunk: {event['data']['chunk']}", flush=True)
    num_events += 1
    if num_events > 30:
        # Truncate the output
        print("...")
        break
```
```output
Chat model chunk: '{'
Parser chunk: {}
Chat model chunk: '\n  '
Chat model chunk: '"'
Chat model chunk: 'countries'
Chat model chunk: '":'
Chat model chunk: ' ['
Parser chunk: {'countries': []}
Chat model chunk: '\n    '
Chat model chunk: '{'
Parser chunk: {'countries': [{}]}
Chat model chunk: '\n      '
Chat model chunk: '"'
Chat model chunk: 'name'
Chat model chunk: '":'
Chat model chunk: ' "'
Parser chunk: {'countries': [{'name': ''}]}
Chat model chunk: 'France'
Parser chunk: {'countries': [{'name': 'France'}]}
Chat model chunk: '",'
Chat model chunk: '\n      '
Chat model chunk: '"'
Chat model chunk: 'population'
Chat model chunk: '":'
Chat model chunk: ' '
Chat model chunk: '67'
Parser chunk: {'countries': [{'name': 'France', 'population': 67}]}
...
```

### 传播回调

:::caution
如果您在工具中使用调用可运行对象，您需要将回调传播到可运行对象；否则，将不会生成任何流事件。
:::

:::note
使用 `RunnableLambdas` 或 `@chain` 装饰器时，回调会在后台自动传播。
:::


```python
from langchain_core.runnables import RunnableLambda
from langchain_core.tools import tool


def reverse_word(word: str):
    return word[::-1]


reverse_word = RunnableLambda(reverse_word)


@tool
def bad_tool(word: str):
    """自定义工具，不传播回调。"""
    return reverse_word.invoke(word)


async for event in bad_tool.astream_events("hello", version="v2"):
    print(event)
```
```output
{'event': 'on_tool_start', 'data': {'input': 'hello'}, 'name': 'bad_tool', 'tags': [], 'run_id': 'ea900472-a8f7-425d-b627-facdef936ee8', 'metadata': {}}
{'event': 'on_chain_start', 'data': {'input': 'hello'}, 'name': 'reverse_word', 'tags': [], 'run_id': '77b01284-0515-48f4-8d7c-eb27c1882f86', 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': 'olleh', 'input': 'hello'}, 'run_id': '77b01284-0515-48f4-8d7c-eb27c1882f86', 'name': 'reverse_word', 'tags': [], 'metadata': {}}
{'event': 'on_tool_end', 'data': {'output': 'olleh'}, 'run_id': 'ea900472-a8f7-425d-b627-facdef936ee8', 'name': 'bad_tool', 'tags': [], 'metadata': {}}
```
这是一个正确传播回调的重新实现。您会注意到现在我们也收到了来自 `reverse_word` 可运行对象的事件。


```python
@tool
def correct_tool(word: str, callbacks):
    """一个正确传播回调的工具。"""
    return reverse_word.invoke(word, {"callbacks": callbacks})


async for event in correct_tool.astream_events("hello", version="v2"):
    print(event)
```
```output
{'event': 'on_tool_start', 'data': {'input': 'hello'}, 'name': 'correct_tool', 'tags': [], 'run_id': 'd5ea83b9-9278-49cc-9f1d-aa302d671040', 'metadata': {}}
{'event': 'on_chain_start', 'data': {'input': 'hello'}, 'name': 'reverse_word', 'tags': [], 'run_id': '44dafbf4-2f87-412b-ae0e-9f71713810df', 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': 'olleh', 'input': 'hello'}, 'run_id': '44dafbf4-2f87-412b-ae0e-9f71713810df', 'name': 'reverse_word', 'tags': [], 'metadata': {}}
{'event': 'on_tool_end', 'data': {'output': 'olleh'}, 'run_id': 'd5ea83b9-9278-49cc-9f1d-aa302d671040', 'name': 'correct_tool', 'tags': [], 'metadata': {}}
```
如果您从 `Runnable Lambdas` 或 `@chains` 中调用可运行对象，则回调将自动传递给您。


```python
from langchain_core.runnables import RunnableLambda


async def reverse_and_double(word: str):
    return await reverse_word.ainvoke(word) * 2


reverse_and_double = RunnableLambda(reverse_and_double)

await reverse_and_double.ainvoke("1234")

async for event in reverse_and_double.astream_events("1234", version="v2"):
    print(event)
```
```output
{'event': 'on_chain_start', 'data': {'input': '1234'}, 'name': 'reverse_and_double', 'tags': [], 'run_id': '03b0e6a1-3e60-42fc-8373-1e7829198d80', 'metadata': {}}
{'event': 'on_chain_start', 'data': {'input': '1234'}, 'name': 'reverse_word', 'tags': [], 'run_id': '5cf26fc8-840b-4642-98ed-623dda28707a', 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': '4321', 'input': '1234'}, 'run_id': '5cf26fc8-840b-4642-98ed-623dda28707a', 'name': 'reverse_word', 'tags': [], 'metadata': {}}
{'event': 'on_chain_stream', 'data': {'chunk': '43214321'}, 'run_id': '03b0e6a1-3e60-42fc-8373-1e7829198d80', 'name': 'reverse_and_double', 'tags': [], 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': '43214321'}, 'run_id': '03b0e6a1-3e60-42fc-8373-1e7829198d80', 'name': 'reverse_and_double', 'tags': [], 'metadata': {}}
```
使用 `@chain` 装饰器：


```python
from langchain_core.runnables import chain


@chain
async def reverse_and_double(word: str):
    return await reverse_word.ainvoke(word) * 2


await reverse_and_double.ainvoke("1234")

async for event in reverse_and_double.astream_events("1234", version="v2"):
    print(event)
```
```output
{'event': 'on_chain_start', 'data': {'input': '1234'}, 'name': 'reverse_and_double', 'tags': [], 'run_id': '1bfcaedc-f4aa-4d8e-beee-9bba6ef17008', 'metadata': {}}
{'event': 'on_chain_start', 'data': {'input': '1234'}, 'name': 'reverse_word', 'tags': [], 'run_id': '64fc99f0-5d7d-442b-b4f5-4537129f67d1', 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': '4321', 'input': '1234'}, 'run_id': '64fc99f0-5d7d-442b-b4f5-4537129f67d1', 'name': 'reverse_word', 'tags': [], 'metadata': {}}
{'event': 'on_chain_stream', 'data': {'chunk': '43214321'}, 'run_id': '1bfcaedc-f4aa-4d8e-beee-9bba6ef17008', 'name': 'reverse_and_double', 'tags': [], 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': '43214321'}, 'run_id': '1bfcaedc-f4aa-4d8e-beee-9bba6ef17008', 'name': 'reverse_and_double', 'tags': [], 'metadata': {}}
```

## 下一步

现在您已经了解了一些使用 LangChain 流式传输最终输出和内部步骤的方法。

要了解更多信息，请查看本节中的其他操作指南，或查看 [Langchain 表达语言的概念指南](/docs/concepts/#langchain-expression-language/)。