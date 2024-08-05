# cassandra-entomology-rag

此模板将通过 CQL（`Cassandra` 向量存储类）使用 Apache Cassandra® 或 Astra DB 执行 RAG

## 环境设置

要进行设置，您需要：
- 一个 [Astra](https://astra.datastax.com) 向量数据库。您必须拥有一个 [数据库管理员令牌](https://awesome-astra.github.io/docs/pages/astra/create-token/#c-procedure)，具体是以 `AstraCS:...` 开头的字符串。
- [数据库 ID](https://awesome-astra.github.io/docs/pages/astra/faq/#where-should-i-find-a-database-identifier)。
- 一个 **OpenAI API 密钥**。 （更多信息 [在这里](https://cassio.org/start_here/#llm-access)）

您也可以使用常规的 Cassandra 集群。在这种情况下，请提供 `USE_CASSANDRA_CLUSTER` 条目，如 `.env.template` 中所示，并提供后续环境变量以指定如何连接到它。

连接参数和密钥必须通过环境变量提供。请参考 `.env.template` 以获取所需变量。

## 用法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package cassandra-entomology-rag
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add cassandra-entomology-rag
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from cassandra_entomology_rag import chain as cassandra_entomology_rag_chain

add_routes(app, cassandra_entomology_rag_chain, path="/cassandra-entomology-rag")
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

如果您在此目录中，则可以直接通过以下方式启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/cassandra-entomology-rag/playground](http://127.0.0.1:8000/cassandra-entomology-rag/playground) 访问游乐场

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/cassandra-entomology-rag")
```

## 参考

独立的仓库与 LangServe 链接： [这里](https://github.com/hemidactylus/langserve_cassandra_entomology_rag)。