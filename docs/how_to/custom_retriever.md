---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/custom_retriever.ipynb
title: 自定义检索器
---

# 如何创建自定义检索器

## 概述

许多 LLM 应用涉及使用 `Retriever` 从外部数据源检索信息。

检索器负责检索与给定用户 `query` 相关的 `Documents` 列表。

检索到的文档通常被格式化为提示，并输入到 LLM 中，使 LLM 能够利用这些信息生成适当的响应（例如，根据知识库回答用户问题）。

## 接口

要创建自己的检索器，您需要扩展 `BaseRetriever` 类并实现以下方法：

| 方法                             | 描述                                              | 必需/可选         |
|----------------------------------|--------------------------------------------------|-------------------|
| `_get_relevant_documents`        | 获取与查询相关的文档。                           | 必需              |
| `_aget_relevant_documents`       | 实现以提供异步原生支持。                         | 可选              |


`_get_relevant_documents` 内部的逻辑可以涉及对数据库或通过请求访问网络的任意调用。

:::tip
通过继承 `BaseRetriever`，您的检索器将自动成为 LangChain [Runnable](/docs/concepts#interface)，并将开箱即用地获得标准的 `Runnable` 功能！
:::


:::info
您可以使用 `RunnableLambda` 或 `RunnableGenerator` 来实现检索器。

将检索器实现为 `BaseRetriever` 而不是 `RunnableLambda`（自定义 [可运行函数](/docs/how_to/functions)）的主要好处是，`BaseRetriever` 是一个众所周知的 LangChain 实体，因此某些监控工具可能会为检索器实现专门的行为。另一个区别是，`BaseRetriever` 在某些 API 中的行为与 `RunnableLambda` 会略有不同；例如，`astream_events` API 中的 `start` 事件将是 `on_retriever_start` 而不是 `on_chain_start`。
:::

## 示例

让我们实现一个玩具检索器，该检索器返回所有文本中包含用户查询文本的文档。

```python
from typing import List

from langchain_core.callbacks import CallbackManagerForRetrieverRun
from langchain_core.documents import Document
from langchain_core.retrievers import BaseRetriever


class ToyRetriever(BaseRetriever):
    """一个玩具检索器，包含包含用户查询的前 k 个文档。

    该检索器仅实现同步方法 _get_relevant_documents。

    如果检索器涉及文件访问或网络访问，它可以受益于 `_aget_relevant_documents` 的原生异步实现。

    和往常一样，对于 Runnables，提供了一个默认的异步实现，该实现委托给在另一个线程上运行的同步实现。
    """

    documents: List[Document]
    """要检索的文档列表。"""
    k: int
    """要返回的前结果数量"""

    def _get_relevant_documents(
        self, query: str, *, run_manager: CallbackManagerForRetrieverRun
    ) -> List[Document]:
        """检索器的同步实现。"""
        matching_documents = []
        for document in documents:
            if len(matching_documents) > self.k:
                return matching_documents

            if query.lower() in document.page_content.lower():
                matching_documents.append(document)
        return matching_documents

    # 可选：通过重写提供更高效的原生实现
    # _aget_relevant_documents
    # async def _aget_relevant_documents(
    #     self, query: str, *, run_manager: AsyncCallbackManagerForRetrieverRun
    # ) -> List[Document]:
    #     """异步获取与查询相关的文档。

    #     参数：
    #         query: 要查找相关文档的字符串
    #         run_manager: 要使用的回调处理程序

    #     返回：
    #         相关文档列表
    #     """
```

## 测试它 🧪


```python
documents = [
    Document(
        page_content="Dogs are great companions, known for their loyalty and friendliness.",
        metadata={"type": "dog", "trait": "loyalty"},
    ),
    Document(
        page_content="Cats are independent pets that often enjoy their own space.",
        metadata={"type": "cat", "trait": "independence"},
    ),
    Document(
        page_content="Goldfish are popular pets for beginners, requiring relatively simple care.",
        metadata={"type": "fish", "trait": "low maintenance"},
    ),
    Document(
        page_content="Parrots are intelligent birds capable of mimicking human speech.",
        metadata={"type": "bird", "trait": "intelligence"},
    ),
    Document(
        page_content="Rabbits are social animals that need plenty of space to hop around.",
        metadata={"type": "rabbit", "trait": "social"},
    ),
]
retriever = ToyRetriever(documents=documents, k=3)
```


```python
retriever.invoke("that")
```



```output
[Document(page_content='Cats are independent pets that often enjoy their own space.', metadata={'type': 'cat', 'trait': 'independence'}),
 Document(page_content='Rabbits are social animals that need plenty of space to hop around.', metadata={'type': 'rabbit', 'trait': 'social'})]
```


这是一个**可运行的**，因此它将受益于标准的可运行接口！🤩


```python
await retriever.ainvoke("that")
```



```output
[Document(page_content='Cats are independent pets that often enjoy their own space.', metadata={'type': 'cat', 'trait': 'independence'}),
 Document(page_content='Rabbits are social animals that need plenty of space to hop around.', metadata={'type': 'rabbit', 'trait': 'social'})]
```



```python
retriever.batch(["dog", "cat"])
```



```output
[[Document(page_content='Dogs are great companions, known for their loyalty and friendliness.', metadata={'type': 'dog', 'trait': 'loyalty'})],
 [Document(page_content='Cats are independent pets that often enjoy their own space.', metadata={'type': 'cat', 'trait': 'independence'})]]
```



```python
async for event in retriever.astream_events("bar", version="v1"):
    print(event)
```
```output
{'event': 'on_retriever_start', 'run_id': 'f96f268d-8383-4921-b175-ca583924d9ff', 'name': 'ToyRetriever', 'tags': [], 'metadata': {}, 'data': {'input': 'bar'}}
{'event': 'on_retriever_stream', 'run_id': 'f96f268d-8383-4921-b175-ca583924d9ff', 'tags': [], 'metadata': {}, 'name': 'ToyRetriever', 'data': {'chunk': []}}
{'event': 'on_retriever_end', 'name': 'ToyRetriever', 'run_id': 'f96f268d-8383-4921-b175-ca583924d9ff', 'tags': [], 'metadata': {}, 'data': {'output': []}}
```

## 贡献

我们欢迎有趣的检索器的贡献！

以下是一个清单，帮助确保你的贡献能够被添加到 LangChain：

文档：

* 检索器包含所有初始化参数的文档字符串，因为这些将在 [API 参考](https://api.python.langchain.com/en/stable/langchain_api_reference.html) 中展示。
* 模型的类文档字符串包含指向检索器所使用的任何相关 API 的链接（例如，如果检索器是从维基百科检索的，最好链接到维基百科 API！）

测试：

* [ ] 添加单元测试或集成测试，以验证 `invoke` 和 `ainvoke` 的工作情况。

优化：

如果检索器连接到外部数据源（例如，API 或文件），那么它几乎肯定会受益于原生异步优化！

* [ ] 提供 `_aget_relevant_documents` 的原生异步实现（由 `ainvoke` 使用）