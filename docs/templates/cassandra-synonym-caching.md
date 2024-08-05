# cassandra-synonym-caching

此模板提供了一个简单的链模板，展示了通过 CQL 使用 Apache Cassandra® 或 Astra DB 支持的 LLM 缓存的用法。

## 环境设置

要设置您的环境，您需要以下内容：

- 一个 [Astra](https://astra.datastax.com) 向量数据库（免费套餐即可！）。**您需要一个 [数据库管理员令牌](https://awesome-astra.github.io/docs/pages/astra/create-token/#c-procedure)**，特别是以 `AstraCS:...` 开头的字符串；
- 同样，准备好您的 [数据库 ID](https://awesome-astra.github.io/docs/pages/astra/faq/#where-should-i-find-a-database-identifier)，您需要在下面输入它；
- 一个 **OpenAI API 密钥**。（更多信息 [在这里](https://cassio.org/start_here/#llm-access)，请注意，默认情况下此演示支持 OpenAI，除非您对代码进行调整。）

_注意：_ 您也可以选择使用常规 Cassandra 集群：为此，请确保提供 `USE_CASSANDRA_CLUSTER` 条目，如 `.env.template` 中所示，并设置后续环境变量以指定如何连接到它。

## 用法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package cassandra-synonym-caching
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add cassandra-synonym-caching
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from cassandra_synonym_caching import chain as cassandra_synonym_caching_chain

add_routes(app, cassandra_synonym_caching_chain, path="/cassandra-synonym-caching")
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

如果您在此目录中，则可以直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行于 
[http://localhost:8000](http://localhost:8000)。

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/cassandra-synonym-caching/playground](http://127.0.0.1:8000/cassandra-synonym-caching/playground) 访问游乐场。

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/cassandra-synonym-caching")
```

## 参考

独立的 LangServe 模板仓库： [这里](https://github.com/hemidactylus/langserve_cassandra_synonym_caching).