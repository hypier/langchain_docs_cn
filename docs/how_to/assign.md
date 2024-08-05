---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/assign.ipynb
sidebar_position: 6
keywords: [RunnablePassthrough, assign, LCEL]
---

# 如何向链的状态添加值

:::info 前提条件

本指南假设您熟悉以下概念：
- [LangChain 表达式语言 (LCEL)](/docs/concepts/#langchain-expression-language)
- [链式可运行任务](/docs/how_to/sequence/)
- [并行调用可运行任务](/docs/how_to/parallel/)
- [自定义函数](/docs/how_to/functions/)
- [数据传递](/docs/how_to/passthrough)

:::

另一种在链的步骤中[传递数据](/docs/how_to/passthrough)的方法是保持链状态的当前值不变，同时在给定的键下分配一个新值。 [`RunnablePassthrough.assign()`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.passthrough.RunnablePassthrough.html#langchain_core.runnables.passthrough.RunnablePassthrough.assign)静态方法接受一个输入值，并添加传递给 assign 函数的额外参数。

这在常见的[LangChain 表达式语言](/docs/concepts/#langchain-expression-language)模式中很有用，该模式是以加法方式创建一个字典，以便用作后续步骤的输入。

以下是一个示例：

```python
%pip install --upgrade --quiet langchain langchain-openai

import os
from getpass import getpass

os.environ["OPENAI_API_KEY"] = getpass()
```

```python
from langchain_core.runnables import RunnableParallel, RunnablePassthrough

runnable = RunnableParallel(
    extra=RunnablePassthrough.assign(mult=lambda x: x["num"] * 3),
    modified=lambda x: x["num"] + 1,
)

runnable.invoke({"num": 1})
```

```output
{'extra': {'num': 1, 'mult': 3}, 'modified': 2}
```

让我们分解一下这里发生的事情。

- 链的输入是 `{"num": 1}`。这个输入被传递给 `RunnableParallel`，它并行调用传入的可运行任务。
- `extra` 键下的值被调用。`RunnablePassthrough.assign()` 保持输入字典中的原始键 (`{"num": 1}`)，并分配一个名为 `mult` 的新键。值为 `lambda x: x["num"] * 3)`，结果为 `3`。因此，结果为 `{"num": 1, "mult": 3}`。
- `{"num": 1, "mult": 3}` 被返回给 `RunnableParallel` 调用，并被设置为键 `extra` 的值。
- 与此同时，`modified` 键被调用。结果为 `2`，因为 lambda 从其输入中提取一个名为 `"num"` 的键并加一。

因此，结果是 `{'extra': {'num': 1, 'mult': 3}, 'modified': 2}`。

## 流式处理

这种方法的一个方便特性是，它允许值在可用时立即通过。为了展示这一点，我们将使用 `RunnablePassthrough.assign()` 立即返回检索链中的源文档：

```python
from langchain_community.vectorstores import FAISS
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI, OpenAIEmbeddings

vectorstore = FAISS.from_texts(
    ["harrison worked at kensho"], embedding=OpenAIEmbeddings()
)
retriever = vectorstore.as_retriever()
template = """Answer the question based only on the following context:
{context}

Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)
model = ChatOpenAI()

generation_chain = prompt | model | StrOutputParser()

retrieval_chain = {
    "context": retriever,
    "question": RunnablePassthrough(),
} | RunnablePassthrough.assign(output=generation_chain)

stream = retrieval_chain.stream("where did harrison work?")

for chunk in stream:
    print(chunk)
```
```output
{'question': 'where did harrison work?'}
{'context': [Document(page_content='harrison worked at kensho')]}
{'output': ''}
{'output': 'H'}
{'output': 'arrison'}
{'output': ' worked'}
{'output': ' at'}
{'output': ' Kens'}
{'output': 'ho'}
{'output': '.'}
{'output': ''}
```
我们可以看到，第一个数据块包含原始的 `"question"`，因为它是立即可用的。第二个数据块包含 `"context"`，因为检索器第二个完成。最后，`generation_chain` 的输出在可用时以数据块的形式流出。

## 下一步

现在您已经了解了如何通过链条传递数据，以帮助格式化流经链条的数据。

要了解更多信息，请参阅本节中有关可运行项的其他操作指南。