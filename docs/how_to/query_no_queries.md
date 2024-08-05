---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/query_no_queries.ipynb
sidebar_position: 3
---

# 如何处理未生成查询的情况

有时，查询分析技术可能允许生成任意数量的查询——包括没有查询！在这种情况下，我们的整体链条需要在决定是否调用检索器之前检查查询分析的结果。

我们将使用模拟数据作为此示例。

## 设置
#### 安装依赖

```python
# %pip install -qU langchain langchain-community langchain-openai langchain-chroma
```

#### 设置环境变量

在这个例子中我们将使用 OpenAI：

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()

# 可选，取消注释以使用 LangSmith 跟踪运行。请在此注册： https://smith.langchain.com.
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

### 创建索引

我们将基于虚假信息创建一个向量存储。

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

texts = ["Harrison worked at Kensho"]
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Chroma.from_texts(
    texts,
    embeddings,
)
retriever = vectorstore.as_retriever()
```

## 查询分析

我们将使用函数调用来结构化输出。然而，我们将配置 LLM，使其在不需要调用表示搜索查询的函数时（如果它决定不这样做）。然后，我们将使用一个提示来进行查询分析，明确说明何时应该以及不应该进行搜索。

```python
from typing import Optional

from langchain_core.pydantic_v1 import BaseModel, Field


class Search(BaseModel):
    """在工作记录数据库上进行搜索。"""

    query: str = Field(
        ...,
        description="应用于工作记录的相似性搜索查询。",
    )
```


```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI

system = """您有能力发出搜索查询以获取帮助回答用户信息的信息。

您不需要查找信息。如果您不需要，请正常回复。"""
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        ("human", "{question}"),
    ]
)
llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
structured_llm = llm.bind_tools([Search])
query_analyzer = {"question": RunnablePassthrough()} | prompt | structured_llm
```

我们可以看到，通过调用这个，我们得到一条消息，有时 - 但不是总是 - 返回一个工具调用。

```python
query_analyzer.invoke("Harrison在哪里工作")
```



```output
AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_ZnoVX4j9Mn8wgChaORyd1cvq', 'function': {'arguments': '{"query":"Harrison"}', 'name': 'Search'}, 'type': 'function'}]})
```



```python
query_analyzer.invoke("你好！")
```



```output
AIMessage(content='你好！我今天能帮您什么？')
```

## 查询分析的检索

那么我们如何将其包含在一个链中呢？让我们看下面的例子。


```python
from langchain_core.output_parsers.openai_tools import PydanticToolsParser
from langchain_core.runnables import chain

output_parser = PydanticToolsParser(tools=[Search])
```


```python
@chain
def custom_chain(question):
    response = query_analyzer.invoke(question)
    if "tool_calls" in response.additional_kwargs:
        query = output_parser.invoke(response)
        docs = retriever.invoke(query[0].query)
        # Could add more logic - like another LLM call - here
        return docs
    else:
        return response
```


```python
custom_chain.invoke("where did Harrison Work")
```
```output
Number of requested results 4 is greater than number of elements in index 1, updating n_results = 1
```


```output
[Document(page_content='Harrison worked at Kensho')]
```



```python
custom_chain.invoke("hi!")
```



```output
AIMessage(content='Hello! How can I assist you today?')
```