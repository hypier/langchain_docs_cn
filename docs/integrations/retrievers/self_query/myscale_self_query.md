---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/self_query/myscale_self_query.ipynb
---

# MyScale

>[MyScale](https://docs.myscale.com/en/) 是一个集成的向量数据库。您可以通过 SQL 访问您的数据库，也可以通过这里的 LangChain 访问。
>`MyScale` 可以利用 [各种数据类型和过滤器功能](https://blog.myscale.com/2023/06/06/why-integrated-database-solution-can-boost-your-llm-apps/#filter-on-anything-without-constraints)。无论您是扩展数据还是将系统扩展到更广泛的应用，它都能提升您的 LLM 应用。

在笔记本中，我们将演示围绕 `MyScale` 向量存储的 `SelfQueryRetriever`，以及我们为 LangChain 贡献的一些额外组件。

简而言之，可以概括为 4 点：
1. 添加 `contain` 比较器，以匹配如果有多个元素匹配的列表
2. 添加 `timestamp` 数据类型以进行日期时间匹配（ISO 格式或 YYYY-MM-DD）
3. 添加 `like` 比较器以进行字符串模式搜索
4. 添加任意函数能力

## 创建 MyScale 向量存储
MyScale 已经与 LangChain 集成了一段时间。因此，您可以按照 [这个笔记本](/docs/integrations/vectorstores/myscale) 创建自己的向量存储，以便进行自查询检索。

**注意：** 所有自查询检索器都需要安装 `lark`（`pip install lark`）。我们使用 `lark` 进行语法定义。在您继续进行下一步之前，我们还想提醒您，需要 `clickhouse-connect` 来与您的 MyScale 后端进行交互。

```python
%pip install --upgrade --quiet  lark clickhouse-connect
```

在本教程中，我们遵循其他示例的设置，并使用 `OpenAIEmbeddings`。请记得获取一个有效访问 LLM 的 OpenAI API 密钥。

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key:")
os.environ["MYSCALE_HOST"] = getpass.getpass("MyScale URL:")
os.environ["MYSCALE_PORT"] = getpass.getpass("MyScale Port:")
os.environ["MYSCALE_USERNAME"] = getpass.getpass("MyScale Username:")
os.environ["MYSCALE_PASSWORD"] = getpass.getpass("MyScale Password:")
```

```python
from langchain_community.vectorstores import MyScale
from langchain_core.documents import Document
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
```

## 创建一些示例数据
正如您所看到的，我们创建的数据与其他自查询检索器相比有一些不同。我们将关键字 `year` 替换为 `date`，这使您可以更精细地控制时间戳。我们还将关键字 `gerne` 的类型更改为字符串列表，其中 LLM 可以使用新的 `contain` 比较器来构造过滤器。我们还为过滤器提供了 `like` 比较器和任意函数支持，这将在接下来的几节中介绍。

现在让我们先看看数据。


```python
docs = [
    Document(
        page_content="A bunch of scientists bring back dinosaurs and mayhem breaks loose",
        metadata={"date": "1993-07-02", "rating": 7.7, "genre": ["science fiction"]},
    ),
    Document(
        page_content="Leo DiCaprio gets lost in a dream within a dream within a dream within a ...",
        metadata={"date": "2010-12-30", "director": "Christopher Nolan", "rating": 8.2},
    ),
    Document(
        page_content="A psychologist / detective gets lost in a series of dreams within dreams within dreams and Inception reused the idea",
        metadata={"date": "2006-04-23", "director": "Satoshi Kon", "rating": 8.6},
    ),
    Document(
        page_content="A bunch of normal-sized women are supremely wholesome and some men pine after them",
        metadata={"date": "2019-08-22", "director": "Greta Gerwig", "rating": 8.3},
    ),
    Document(
        page_content="Toys come alive and have a blast doing so",
        metadata={"date": "1995-02-11", "genre": ["animated"]},
    ),
    Document(
        page_content="Three men walk into the Zone, three men walk out of the Zone",
        metadata={
            "date": "1979-09-10",
            "director": "Andrei Tarkovsky",
            "genre": ["science fiction", "adventure"],
            "rating": 9.9,
        },
    ),
]
vectorstore = MyScale.from_documents(
    docs,
    embeddings,
)
```

## 创建自查询检索器
就像其他检索器一样……简单而美好。

```python
from langchain.chains.query_constructor.base import AttributeInfo
from langchain.retrievers.self_query.base import SelfQueryRetriever
from langchain_openai import ChatOpenAI

metadata_field_info = [
    AttributeInfo(
        name="genre",
        description="电影的类型。"
        "只支持相等和包含比较。"
        "以下是一些示例: genre = [' A '], genre = [' A ', 'B'], contain (genre, 'A')",
        type="list[string]",
    ),
    # 如果您想包含列表的长度，只需将其定义为新列
    # 这将教LLM在构建过滤器时将其用作列。
    AttributeInfo(
        name="length(genre)",
        description="电影类型的数量",
        type="integer",
    ),
    # 现在您可以将列定义为时间戳。只需将类型设置为时间戳即可。
    AttributeInfo(
        name="date",
        description="电影上映的日期",
        type="timestamp",
    ),
    AttributeInfo(
        name="director",
        description="电影导演的姓名",
        type="string",
    ),
    AttributeInfo(
        name="rating", description="电影的1-10评分", type="float"
    ),
]
document_content_description = "电影的简要概述"
llm = ChatOpenAI(temperature=0, model_name="gpt-4o")
retriever = SelfQueryRetriever.from_llm(
    llm, vectorstore, document_content_description, metadata_field_info, verbose=True
)
```

## 测试自查询检索器现有功能
现在我们可以尝试实际使用我们的检索器！


```python
# This example only specifies a relevant query
retriever.invoke("What are some movies about dinosaurs")
```


```python
# This example only specifies a filter
retriever.invoke("I want to watch a movie rated higher than 8.5")
```


```python
# This example specifies a query and a filter
retriever.invoke("Has Greta Gerwig directed any movies about women")
```


```python
# This example specifies a composite filter
retriever.invoke("What's a highly rated (above 8.5) science fiction film?")
```


```python
# This example specifies a query and composite filter
retriever.invoke(
    "What's a movie after 1990 but before 2005 that's all about toys, and preferably is animated"
)
```

# 等一下... 还有什么？

使用 MyScale 的自查询检索器可以做更多事情！让我们来看看。

```python
# You can use length(genres) to do anything you want
retriever.invoke("What's a movie that have more than 1 genres?")
```

```python
# Fine-grained datetime? You got it already.
retriever.invoke("What's a movie that release after feb 1995?")
```

```python
# Don't know what your exact filter should be? Use string pattern match!
retriever.invoke("What's a movie whose name is like Andrei?")
```

```python
# Contain works for lists: so you can match a list with contain comparator!
retriever.invoke("What's a movie who has genres science fiction and adventure?")
```

## 过滤 k

我们还可以使用自查询检索器来指定 `k`：要获取的文档数量。

我们可以通过将 `enable_limit=True` 传递给构造函数来实现这一点。


```python
retriever = SelfQueryRetriever.from_llm(
    llm,
    vectorstore,
    document_content_description,
    metadata_field_info,
    enable_limit=True,
    verbose=True,
)
```


```python
# 此示例仅指定了一个相关查询
retriever.invoke("what are two movies about dinosaurs")
```