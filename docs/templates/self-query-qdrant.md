# self-query-qdrant

此模板使用 Qdrant 和 OpenAI 执行 [自查询](https://python.langchain.com/docs/modules/data_connection/retrievers/self_query/)。默认情况下，它使用一个包含 10 个文档的人工数据集，但您可以用自己的数据集替换它。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

将 `QDRANT_URL` 设置为您的 Qdrant 实例的 URL。如果您使用 [Qdrant Cloud](https://cloud.qdrant.io)，您还需要设置 `QDRANT_API_KEY` 环境变量。如果您没有设置其中任何一个，模板将尝试连接本地 Qdrant 实例，地址为 `http://localhost:6333`。

```shell
export QDRANT_URL=
export QDRANT_API_KEY=

export OPENAI_API_KEY=
```

## 使用

要使用此包，首先安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

创建一个新的 LangChain 项目，并将此包作为唯一的包安装：

```shell
langchain app new my-app --package self-query-qdrant
```

要将其添加到现有项目中，请运行：

```shell
langchain app add self-query-qdrant
```

### 默认设置

在启动服务器之前，您需要创建一个 Qdrant 集合并对文档进行索引。可以通过运行以下命令来完成：

```python
from self_query_qdrant.chain import initialize

initialize()
```

将以下代码添加到您的 `app/server.py` 文件中：

```python
from self_query_qdrant.chain import chain

add_routes(app, chain, path="/self-query-qdrant")
```

默认数据集包含关于菜肴的 10 个文档，以及它们的价格和餐厅信息。您可以在 `packages/self-query-qdrant/self_query_qdrant/defaults.py` 文件中找到这些文档。以下是其中一个文档：

```python
from langchain_core.documents import Document

Document(
    page_content="Spaghetti with meatballs and tomato sauce",
    metadata={
        "price": 12.99,
        "restaurant": {
            "name": "Olive Garden",
            "location": ["New York", "Chicago", "Los Angeles"],
        },
    },
)
```

自查询功能允许对文档执行语义搜索，并根据元数据进行一些额外的过滤。例如，您可以搜索价格低于 $15 且在纽约提供的菜肴。

### 自定义

上述所有示例假设您希望使用默认设置启动模板。如果您想自定义模板，可以通过将参数传递给 `create_chain` 函数来实现，函数位于 `app/server.py` 文件中：

```python
from langchain_community.llms import Cohere
from langchain_community.embeddings import HuggingFaceEmbeddings
from langchain.chains.query_constructor.schema import AttributeInfo

from self_query_qdrant.chain import create_chain

chain = create_chain(
    llm=Cohere(),
    embeddings=HuggingFaceEmbeddings(),
    document_contents="Descriptions of cats, along with their names and breeds.",
    metadata_field_info=[
        AttributeInfo(name="name", description="Name of the cat", type="string"),
        AttributeInfo(name="breed", description="Cat's breed", type="string"),
    ],
    collection_name="cats",
)
```

对于创建 Qdrant 集合并索引文档的 `initialize` 函数也是如此：

```python
from langchain_core.documents import Document
from langchain_community.embeddings import HuggingFaceEmbeddings

from self_query_qdrant.chain import initialize

initialize(
    embeddings=HuggingFaceEmbeddings(),
    collection_name="cats",
    documents=[
        Document(
            page_content="A mean lazy old cat who destroys furniture and eats lasagna",
            metadata={"name": "Garfield", "breed": "Tabby"},
        ),
        ...
    ]
)
```

该模板灵活，可以轻松用于不同文档集。

### LangSmith

（可选）如果您可以访问 LangSmith，请配置它以帮助追踪、监控和调试 LangChain 应用程序。如果您没有访问权限，请跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接通过以下命令启动 LangServe 实例：

```shell
langchain serve
```

### 本地服务器

这将启动在本地运行的 FastAPI 应用，地址为 
[http://localhost:8000](http://localhost:8000)

您可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
访问游乐场请前往 [http://127.0.0.1:8000/self-query-qdrant/playground](http://127.0.0.1:8000/self-query-qdrant/playground)

通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/self-query-qdrant")
```