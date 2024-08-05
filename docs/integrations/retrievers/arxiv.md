---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/arxiv.ipynb
sidebar_label: Arxiv
---

# ArxivRetriever

## 概述

>[arXiv](https://arxiv.org/) 是一个开放获取的档案，包含200万篇学术文章，涵盖物理学、数学、计算机科学、定量生物学、定量金融、统计学、电气工程与系统科学以及经济学等领域。

本笔记本展示了如何将来自 Arxiv.org 的科学文章检索为下游使用的 [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html) 格式。

有关所有 `ArxivRetriever` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.arxiv.ArxivRetriever.html)。

### 集成细节

| 检索器 | 来源 | 包 |
| :--- | :--- | :---: |
[ArxivRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.arxiv.ArxivRetriever.html) | [arxiv.org](https://arxiv.org/)上的学术文章 | langchain_community |

## 设置

如果您想从单个查询中获取自动跟踪，您还可以通过取消注释以下内容来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：

```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### 安装

该检索器位于 `langchain-community` 包中。我们还需要 [arxiv](https://pypi.org/project/arxiv/) 依赖项：

```python
%pip install -qU langchain-community arxiv
```

## 实例化

`ArxivRetriever` 参数包括：
- 可选的 `load_max_docs`：默认值为 100。用于限制下载的文档数量。下载所有 100 个文档需要时间，因此在实验中使用较小的数字。目前有一个硬性限制为 300。
- 可选的 `load_all_available_meta`：默认值为 False。默认情况下仅下载最重要的字段：`Published`（文档发布/最后更新的日期）、`Title`、`Authors`、`Summary`。如果为 True，则还会下载其他字段。
- `get_full_documents`：布尔值，默认值为 False。决定是否获取文档的全文。

有关更多详细信息，请参见 [API 参考](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.arxiv.ArxivRetriever.html)。

```python
from langchain_community.retrievers import ArxivRetriever

retriever = ArxivRetriever(
    load_max_docs=2,
    get_ful_documents=True,
)
```

## 用法

`ArxivRetriever` 支持通过文章标识符进行检索：

```python
docs = retriever.invoke("1605.08386")
```

```python
docs[0].metadata  # 文档的元信息
```

```output
{'Entry ID': 'http://arxiv.org/abs/1605.08386v1',
 'Published': datetime.date(2016, 5, 26),
 'Title': 'Heat-bath random walks with Markov bases',
 'Authors': 'Caprice Stanley, Tobias Windisch'}
```

```python
docs[0].page_content[:400]  # 文档的内容
```

```output
'Graphs on lattice points are studied whose edges come from a finite set of\nallowed moves of arbitrary length. We show that the diameter of these graphs on\nfibers of a fixed integer matrix can be bounded from above by a constant. We\nthen study the mixing behaviour of heat-bath random walks on these graphs. We\nalso state explicit conditions on the set of moves so that the heat-bath random\nwalk, a ge'
```

`ArxivRetriever` 还支持基于自然语言文本的检索：

```python
docs = retriever.invoke("What is the ImageBind model?")
```

```python
docs[0].metadata
```

```output
{'Entry ID': 'http://arxiv.org/abs/2305.05665v2',
 'Published': datetime.date(2023, 5, 31),
 'Title': 'ImageBind: One Embedding Space To Bind Them All',
 'Authors': 'Rohit Girdhar, Alaaeldin El-Nouby, Zhuang Liu, Mannat Singh, Kalyan Vasudev Alwala, Armand Joulin, Ishan Misra'}
```

## 在链中使用

与其他检索器一样，`ArxivRetriever` 可以通过 [chains](/docs/how_to/sequence/) 集成到 LLM 应用程序中。

我们需要一个 LLM 或聊天模型：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />


```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough

prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the context provided.

Context: {context}

Question: {question}"""
)


def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)


chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
```


```python
chain.invoke("What is the ImageBind model?")
```



```output
'The ImageBind model is an approach to learn a joint embedding across six different modalities - images, text, audio, depth, thermal, and IMU data. It shows that only image-paired data is sufficient to bind the modalities together and can leverage large scale vision-language models for zero-shot capabilities and emergent applications such as cross-modal retrieval, composing modalities with arithmetic, cross-modal detection and generation.'
```

## API 参考

有关所有 `ArxivRetriever` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.arxiv.ArxivRetriever.html)。

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)