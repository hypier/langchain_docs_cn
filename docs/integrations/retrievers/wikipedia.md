---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/wikipedia.ipynb
sidebar_label: 维基百科
---

# WikipediaRetriever

## 概述
>[Wikipedia](https://wikipedia.org/) 是一个多语言的免费在线百科全书，由志愿者社区（称为维基人）通过开放协作和使用名为 MediaWiki 的基于维基的编辑系统编写和维护。`Wikipedia` 是历史上最大的、阅读量最高的参考作品。

本笔记本展示了如何将 `wikipedia.org` 的维基页面检索到下游使用的 [Document](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html) 格式中。

### 集成细节

| 检索器 | 来源 | 包 |
| :--- | :--- | :---: |
[WikipediaRetriever](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.wikipedia.WikipediaRetriever.html) | [维基百科](https://www.wikipedia.org/) 文章 | langchain_community |

## 设置
如果您想从单个工具的运行中获得自动追踪，可以通过取消注释以下内容来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：


```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### 安装

该集成位于 `langchain-community` 包中。我们还需要安装 `wikipedia` python 包本身。

```python
%pip install -qU langchain_community wikipedia
```

## 实例化

现在我们可以实例化我们的检索器：

`WikipediaRetriever` 参数包括：
- 可选的 `lang`：默认值为 "en"。用于在特定语言的维基百科部分进行搜索
- 可选的 `load_max_docs`：默认值为 100。用于限制下载文档的数量。下载所有 100 个文档需要时间，因此在实验中使用较小的数字。目前有一个硬性限制为 300。
- 可选的 `load_all_available_meta`：默认值为 False。默认情况下，仅下载最重要的字段：`Published`（文档发布/最后更新的日期）、`title`、`Summary`。如果为 True，则还会下载其他字段。

`get_relevant_documents()` 有一个参数 `query`：用于在维基百科中查找文档的自由文本


```python
from langchain_community.retrievers import WikipediaRetriever

retriever = WikipediaRetriever()
```

## 用法


```python
docs = retriever.invoke("TOKYO GHOUL")
```


```python
print(docs[0].page_content[:400])
```
```output
Tokyo Ghoul (Japanese: 東京喰種（トーキョーグール）, Hepburn: Tōkyō Gūru) is a Japanese dark fantasy manga series written and illustrated by Sui Ishida. It was serialized in Shueisha's seinen manga magazine Weekly Young Jump from September 2011 to September 2014, with its chapters collected in 14 tankōbon volumes. The story is set in an alternate version of Tokyo where humans coexist with ghouls, beings who loo
```

## 在链中使用
与其他检索器一样，`WikipediaRetriever` 可以通过 [chains](/docs/how_to/sequence/) 集成到 LLM 应用程序中。

我们需要一个 LLM 或聊天模型：

import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />


```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough

prompt = ChatPromptTemplate.from_template(
    """
    Answer the question based only on the context provided.
    Context: {context}
    Question: {question}
    """
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
chain.invoke(
    "Who is the main character in `Tokyo Ghoul` and does he transform into a ghoul?"
)
```



```output
'The main character in Tokyo Ghoul is Ken Kaneki, who transforms into a ghoul after receiving an organ transplant from a ghoul named Rize.'
```

## API 参考

有关所有 `WikipediaRetriever` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/retrievers/langchain_community.retrievers.wikipedia.WikipediaRetriever.html#langchain-community-retrievers-wikipedia-wikipediaretriever)。

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)