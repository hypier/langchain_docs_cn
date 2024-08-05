# neo4j-cypher-memory

此模板允许您使用 OpenAI LLM 与 Neo4j 图形数据库进行自然语言对话。它将自然语言问题转换为 Cypher 查询（用于从 Neo4j 数据库中提取数据），执行查询，并根据查询结果提供自然语言响应。此外，它还具有一个对话记忆模块，将对话历史存储在 Neo4j 图形数据库中。对话记忆为每个用户会话独立维护，确保个性化的互动。为此，请在使用对话链时提供 `user_id` 和 `session_id`。

## 环境设置

定义以下环境变量：

```
OPENAI_API_KEY=<YOUR_OPENAI_API_KEY>
NEO4J_URI=<YOUR_NEO4J_URI>
NEO4J_USERNAME=<YOUR_NEO4J_USERNAME>
NEO4J_PASSWORD=<YOUR_NEO4J_PASSWORD>
```

## Neo4j 数据库设置

有多种方法可以设置 Neo4j 数据库。

### Neo4j Aura

Neo4j AuraDB 是一个完全托管的云图数据库服务。
在 [Neo4j Aura](https://neo4j.com/cloud/platform/aura-graph-database?utm_source=langchain&utm_content=langserve) 上创建一个免费的实例。
当您启动一个免费的数据库实例时，您将收到访问数据库的凭据。

## 用数据填充

如果您想用一些示例数据填充数据库，可以运行 `python ingest.py`。  
该脚本将用示例电影数据填充数据库。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此包作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package neo4j-cypher-memory
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add neo4j-cypher-memory
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from neo4j_cypher_memory import chain as neo4j_cypher_memory_chain

add_routes(app, neo4j_cypher_memory_chain, path="/neo4j-cypher-memory")
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

如果您在该目录中，则可以直接通过以下方式启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/neo4j_cypher_memory/playground](http://127.0.0.1:8000/neo4j_cypher/playground) 访问游乐场。

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/neo4j-cypher-memory")
```