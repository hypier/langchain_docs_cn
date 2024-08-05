---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/sequence.ipynb
keywords: [可运行的, 可运行项, 可运行序列, LCEL, 链, 链条, 链接]
---

# 如何链式调用可运行对象

:::info 前提条件

本指南假设您对以下概念有一定了解：
- [LangChain 表达式语言 (LCEL)](/docs/concepts/#langchain-expression-language)
- [提示模板](/docs/concepts/#prompt-templates)
- [聊天模型](/docs/concepts/#chat-models)
- [输出解析器](/docs/concepts/#output-parsers)

:::

关于 [LangChain 表达式语言](/docs/concepts/#langchain-expression-language) 的一个要点是，任何两个可运行对象都可以“链式”组合成序列。前一个可运行对象的 `.invoke()` 调用的输出作为输入传递给下一个可运行对象。这可以使用管道操作符 (`|`) 或更明确的 `.pipe()` 方法来完成，二者功能相同。

生成的 [`RunnableSequence`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableSequence.html) 本身也是一个可运行对象，这意味着它可以像其他任何可运行对象一样被调用、流式传输或进一步链式调用。以这种方式链式调用可运行对象的优点包括高效的流式传输（序列将尽快流式输出可用的内容）以及使用 [LangSmith](/docs/how_to/debugging) 等工具进行调试和追踪。

## 管道操作符： `|`

为了展示这个是如何工作的，让我们通过一个示例来讲解。我们将通过 LangChain 中的一个常见模式：使用 [提示模板](/docs/how_to#prompt-templates) 将输入格式化为 [聊天模型](/docs/how_to#chat-models)，最后将聊天消息输出转换为字符串，使用 [输出解析器](/docs/how_to#output-parsers)。

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs
  customVarName="model"
/>


```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_template("tell me a joke about {topic}")

chain = prompt | model | StrOutputParser()
```

提示和模型都是可运行的，提示调用的输出类型与聊天模型的输入类型相同，因此我们可以将它们串联在一起。然后我们可以像其他可运行的内容一样调用结果序列：


```python
chain.invoke({"topic": "bears"})
```



```output
"Here's a bear joke for you:\n\nWhy did the bear dissolve in water?\nBecause it was a polar bear!"
```

### 强制转换

我们甚至可以将这个链与更多的可运行对象结合起来，创建另一个链。这可能涉及使用其他类型的可运行对象进行一些输入/输出格式化，具体取决于链组件所需的输入和输出。

例如，假设我们想要将笑话生成链与另一个链组合，该链评估生成的笑话是否有趣。

我们需要小心如何将输入格式化到下一个链中。在下面的示例中，链中的字典会被自动解析并转换为[`RunnableParallel`](/docs/how_to/parallel)，它并行运行所有值并返回一个包含结果的字典。

这恰好是下一个提示模板所期望的格式。以下是它的实际应用：

```python
from langchain_core.output_parsers import StrOutputParser

analysis_prompt = ChatPromptTemplate.from_template("is this a funny joke? {joke}")

composed_chain = {"joke": chain} | analysis_prompt | model | StrOutputParser()

composed_chain.invoke({"topic": "bears"})
```

```output
'Haha, that\'s a clever play on words! Using "polar" to imply the bear dissolved or became polar/polarized when put in water. Not the most hilarious joke ever, but it has a cute, groan-worthy pun that makes it mildly amusing. I appreciate a good pun or wordplay joke.'
```

函数也会被强制转换为可运行对象，因此您也可以向链中添加自定义逻辑。下面的链与之前的逻辑流程相同：

```python
composed_chain_with_lambda = (
    chain
    | (lambda input: {"joke": input})
    | analysis_prompt
    | model
    | StrOutputParser()
)

composed_chain_with_lambda.invoke({"topic": "beets"})
```

```output
"Haha, that's a cute and punny joke! I like how it plays on the idea of beets blushing or turning red like someone blushing. Food puns can be quite amusing. While not a total knee-slapper, it's a light-hearted, groan-worthy dad joke that would make me chuckle and shake my head. Simple vegetable humor!"
```

然而，请记住，像这样使用函数可能会干扰流式操作。有关更多信息，请参见[本节](/docs/how_to/functions)。

## `.pipe()` 方法

我们也可以使用 `.pipe()` 方法来组合相同的序列。它的样子如下：


```python
from langchain_core.runnables import RunnableParallel

composed_chain_with_pipe = (
    RunnableParallel({"joke": chain})
    .pipe(analysis_prompt)
    .pipe(model)
    .pipe(StrOutputParser())
)

composed_chain_with_pipe.invoke({"topic": "battlestar galactica"})
```



```output
"I cannot reproduce any copyrighted material verbatim, but I can try to analyze the humor in the joke you provided without quoting it directly.\n\nThe joke plays on the idea that the Cylon raiders, who are the antagonists in the Battlestar Galactica universe, failed to locate the human survivors after attacking their home planets (the Twelve Colonies) due to using an outdated and poorly performing operating system (Windows Vista) for their targeting systems.\n\nThe humor stems from the juxtaposition of a futuristic science fiction setting with a relatable real-world frustration – the use of buggy, slow, or unreliable software or technology. It pokes fun at the perceived inadequacies of Windows Vista, which was widely criticized for its performance issues and other problems when it was released.\n\nBy attributing the Cylons' failure to locate the humans to their use of Vista, the joke creates an amusing and unexpected connection between a fictional advanced race of robots and a familiar technological annoyance experienced by many people in the real world.\n\nOverall, the joke relies on incongruity and relatability to generate humor, but without reproducing any copyrighted material directly."
```


或者简化版：


```python
composed_chain_with_pipe = RunnableParallel({"joke": chain}).pipe(
    analysis_prompt, model, StrOutputParser()
)
```

## 相关

- [流媒体](/docs/how_to/streaming/): 查看流媒体指南以了解链的流媒体行为