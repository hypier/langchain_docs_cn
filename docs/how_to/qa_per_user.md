---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/qa_per_user.ipynb
---

# 如何进行用户级检索

本指南演示如何配置检索链的运行时属性。一个示例应用是根据用户限制检索器可用的文档。

在构建检索应用时，您通常需要考虑多个用户。这意味着您可能不仅仅为一个用户存储数据，而是为许多不同的用户存储数据，并且他们不应该能够看到彼此的数据。这意味着您需要能够配置检索链，以仅检索特定信息。这通常涉及两个步骤。

**步骤 1：确保您使用的检索器支持多个用户**

目前，LangChain 中没有统一的标志或过滤器。相反，每个向量存储和检索器可能都有自己的支持，并且可能称为不同的名称（命名空间、多租户等）。对于向量存储，这通常作为关键字参数在 `similarity_search` 时传入。通过阅读文档或源代码，确定您使用的检索器是否支持多个用户，如果支持，如何使用它。

注意：为不支持（或未记录）多个用户的检索器添加文档和/或支持是对 LangChain 的一个很好的贡献方式。

**步骤 2：将该参数添加为链的可配置字段**

这将使您能够轻松调用链并在运行时配置任何相关标志。有关配置的更多信息，请参阅 [此文档](/docs/how_to/configure)。

现在，在运行时，您可以使用可配置字段调用此链。

## 代码示例

让我们看看在代码中这是什么样子的具体示例。我们将使用 Pinecone 作为这个示例。

要配置 Pinecone，请设置以下环境变量：

- `PINECONE_API_KEY`: 您的 Pinecone API 密钥


```python
from langchain_openai import OpenAIEmbeddings
from langchain_pinecone import PineconeVectorStore

embeddings = OpenAIEmbeddings()
vectorstore = PineconeVectorStore(index_name="test-example", embedding=embeddings)

vectorstore.add_texts(["i worked at kensho"], namespace="harrison")
vectorstore.add_texts(["i worked at facebook"], namespace="ankush")
```



```output
['ce15571e-4e2f-44c9-98df-7e83f6f63095']
```


`namespace` 的 pinecone kwarg 可用于分隔文档


```python
# This will only get documents for Ankush
vectorstore.as_retriever(search_kwargs={"namespace": "ankush"}).get_relevant_documents(
    "where did i work?"
)
```



```output
[Document(page_content='i worked at facebook')]
```



```python
# This will only get documents for Harrison
vectorstore.as_retriever(
    search_kwargs={"namespace": "harrison"}
).get_relevant_documents("where did i work?")
```



```output
[Document(page_content='i worked at kensho')]
```


我们现在可以创建将用于问答的链。

让我们首先选择一个 LLM。
import ChatModelTabs from "@theme/ChatModelTabs";

<ChatModelTabs customVarName="llm" />

这是基本的问答链设置。


```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import (
    ConfigurableField,
    RunnablePassthrough,
)

template = """Answer the question based only on the following context:
{context}
Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)

retriever = vectorstore.as_retriever()
```

在这里，我们将检索器标记为具有可配置字段。所有 vectorstore 检索器都有 `search_kwargs` 作为字段。这只是一个字典，包含特定于 vectorstore 的字段。

这将使我们在调用链时传入 `search_kwargs` 的值。


```python
configurable_retriever = retriever.configurable_fields(
    search_kwargs=ConfigurableField(
        id="search_kwargs",
        name="Search Kwargs",
        description="The search kwargs to use",
    )
)
```

我们现在可以使用可配置检索器创建链


```python
chain = (
    {"context": configurable_retriever, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)
```

我们现在可以使用可配置选项调用链。`search_kwargs` 是可配置字段的 id。值是用于 Pinecone 的搜索 kwargs


```python
chain.invoke(
    "where did the user work?",
    config={"configurable": {"search_kwargs": {"namespace": "harrison"}}},
)
```



```output
'The user worked at Kensho.'
```



```python
chain.invoke(
    "where did the user work?",
    config={"configurable": {"search_kwargs": {"namespace": "ankush"}}},
)
```



```output
'The user worked at Facebook.'
```


有关多用户的更多 vectorstore 实现，请参阅特定页面，例如 [Milvus](/docs/integrations/vectorstores/milvus).