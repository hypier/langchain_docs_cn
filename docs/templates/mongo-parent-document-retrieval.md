# mongo-parent-document-retrieval

此模板使用 MongoDB 和 OpenAI 执行 RAG。
它执行一种更高级的 RAG 形式，称为父文档检索。

在这种检索形式中，首先将大型文档拆分为中等大小的块。
然后，这些中等大小的块被拆分为小块。
为小块创建嵌入。
当查询到来时，为该查询创建一个嵌入并与小块进行比较。
但不是将小块直接传递给 LLM 进行生成，而是传递来自小块的中等大小块。
这有助于实现更细粒度的搜索，同时传递更大的上下文（在生成过程中可能会有用）。

## 环境设置

您应该导出两个环境变量，一个是您的 MongoDB URI，另一个是您的 OpenAI API KEY。如果您没有 MongoDB URI，请参阅底部的 `设置 Mongo` 部分以获取相关指示。

```shell
export MONGO_URI=...
export OPENAI_API_KEY=...
```

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package mongo-parent-document-retrieval
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add mongo-parent-document-retrieval
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from mongo_parent_document_retrieval import chain as mongo_parent_document_retrieval_chain

add_routes(app, mongo_parent_document_retrieval_chain, path="/mongo-parent-document-retrieval")
```

（可选）现在让我们配置 LangSmith。 
LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。 
您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。 
如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您尚未连接到 Mongo 搜索索引，请在继续之前查看下面的 `MongoDB 设置` 部分。
请注意，由于父文档检索使用不同的索引策略，因此您可能希望运行此新的设置。

如果您有要连接的 MongoDB 搜索索引，请在 `mongo_parent_document_retrieval/chain.py` 中编辑连接详细信息。

如果您在此目录中，则可以直接通过以下命令启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/mongo-parent-document-retrieval/playground](http://127.0.0.1:8000/mongo-parent-document-retrieval/playground) 访问游乐场。

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/mongo-parent-document-retrieval")
```

有关更多上下文，请参阅 [此笔记本](https://colab.research.google.com/drive/1cr2HBAHyBmwKUerJq2if0JaNhy-hIq7I#scrollTo=TZp7_CBfxTOB)。

## MongoDB 设置

如果您需要设置 MongoDB 帐户并导入数据，请使用此步骤。  
我们将首先按照标准的 MongoDB Atlas 设置说明 [这里](https://www.mongodb.com/docs/atlas/getting-started/) 进行操作。

1. 创建一个帐户（如果尚未完成）  
2. 创建一个新项目（如果尚未完成）  
3. 找到您的 MongoDB URI。

这可以通过访问部署概览页面并连接到您的数据库来完成。

接下来，我们查看可用的驱动程序。

其中我们将看到我们的 URI 列出。

然后将其设置为本地环境变量：

```shell
export MONGO_URI=...
```

4. 我们还将为 OpenAI 设置一个环境变量（我们将其用作 LLM）。

```shell
export OPENAI_API_KEY=...
```

5. 现在让我们导入一些数据！我们可以通过进入此目录并运行 `ingest.py` 中的代码来实现，例如：

```shell
python ingest.py
```

请注意，您可以（并且应该！）更改此内容以导入您选择的数据。

6. 现在我们需要在我们的数据上设置一个向量索引。

我们可以首先连接到我们的数据库所在的集群。

然后我们可以导航到列出所有集合的地方。

接下来，我们可以找到我们想要的集合，并查看该集合的搜索索引。

这可能是空的，我们想要创建一个新的索引：

我们将使用 JSON 编辑器来创建它。

然后我们将粘贴以下 JSON：

```text
{
  "mappings": {
    "dynamic": true,
    "fields": {
      "doc_level": [
        {
          "type": "token"
        }
      ],
      "embedding": {
        "dimensions": 1536,
        "similarity": "cosine",
        "type": "knnVector"
      }
    }
  }
}
```

从那里，点击“下一步”，然后点击“创建搜索索引”。这会花费一些时间，但您应该会在您的数据上拥有一个索引！