---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/query_multiple_queries.ipynb
sidebar_position: 4
---

# 如何在查询分析中处理多个查询

有时，查询分析技术可能会生成多个查询。在这种情况下，我们需要记住运行所有查询，然后将结果合并。我们将展示一个简单的示例（使用模拟数据）来说明如何做到这一点。

## 设置
#### 安装依赖项


```python
# %pip install -qU langchain langchain-community langchain-openai langchain-chroma
```

#### 设置环境变量

在这个示例中我们将使用 OpenAI：


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()

# 可选，取消注释以使用 LangSmith 跟踪运行。在此注册： https://smith.langchain.com。
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

### 创建索引

我们将基于虚假信息创建一个向量存储。

```python
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_text_splitters import RecursiveCharacterTextSplitter

texts = ["Harrison worked at Kensho", "Ankush worked at Facebook"]
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Chroma.from_texts(
    texts,
    embeddings,
)
retriever = vectorstore.as_retriever(search_kwargs={"k": 1})
```

## 查询分析

我们将使用函数调用来构造输出。我们将允许它返回多个查询。

```python
from typing import List, Optional

from langchain_core.pydantic_v1 import BaseModel, Field


class Search(BaseModel):
    """在职位记录数据库中进行搜索。"""

    queries: List[str] = Field(
        ...,
        description="要搜索的不同查询",
    )
```

```python
from langchain_core.output_parsers.openai_tools import PydanticToolsParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI

output_parser = PydanticToolsParser(tools=[Search])

system = """您可以发出搜索查询以获取信息，以帮助回答用户的信息。

如果您需要查找两个不同的信息，您是可以这样做的！"""
prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system),
        ("human", "{question}"),
    ]
)
llm = ChatOpenAI(model="gpt-3.5-turbo-0125", temperature=0)
structured_llm = llm.with_structured_output(Search)
query_analyzer = {"question": RunnablePassthrough()} | prompt | structured_llm
```
```output
/Users/harrisonchase/workplace/langchain/libs/core/langchain_core/_api/beta_decorator.py:86: LangChainBetaWarning: The function `with_structured_output` is in beta. It is actively being worked on, so the API may change.
  warn_beta(
```
我们可以看到这允许创建多个查询。

```python
query_analyzer.invoke("where did Harrison Work")
```

```output
Search(queries=['Harrison work location'])
```

```python
query_analyzer.invoke("where did Harrison and ankush Work")
```

```output
Search(queries=['Harrison work place', 'Ankush work place'])
```

## 带查询分析的检索

那么我们如何将其包含在链中呢？一个可以使这变得更容易的事情是，如果我们异步调用我们的检索器——这将使我们能够循环遍历查询，而不会因响应时间而被阻塞。

```python
from langchain_core.runnables import chain
```

```python
@chain
async def custom_chain(question):
    response = await query_analyzer.ainvoke(question)
    docs = []
    for query in response.queries:
        new_docs = await retriever.ainvoke(query)
        docs.extend(new_docs)
    # 你可能想在这里考虑对文档进行重新排序或去重
    # 但这是一个单独的话题
    return docs
```

```python
await custom_chain.ainvoke("where did Harrison Work")
```

```output
[Document(page_content='Harrison worked at Kensho')]
```

```python
await custom_chain.ainvoke("where did Harrison and ankush Work")
```

```output
[Document(page_content='Harrison worked at Kensho'),
 Document(page_content='Ankush worked at Facebook')]
```