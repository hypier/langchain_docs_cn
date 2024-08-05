# neo4j-semantic-ollama

此模板旨在实现一个能够通过语义层与图形数据库（如 Neo4j）交互的代理，使用 Mixtral 作为基于 JSON 的代理。
语义层为代理提供了一套强大的工具，使其能够根据用户的意图与图形数据库进行交互。
在[相关博客文章](https://medium.com/towards-data-science/enhancing-interaction-between-language-models-and-graph-databases-via-a-semantic-layer-0a78ad3eba49)中了解更多关于语义层模板的信息，以及关于[与 Ollama 一起使用的 Mixtral 代理](https://blog.langchain.dev/json-based-agents-with-ollama-and-langchain/)的具体内容。

## 工具

该代理利用多个工具有效地与 Neo4j 图形数据库进行交互：

1. **信息工具**：
   - 检索有关电影或个人的数据，确保代理能够访问最新和最相关的信息。
2. **推荐工具**：
   - 根据用户的偏好和输入提供电影推荐。
3. **记忆工具**：
   - 在知识图谱中存储用户偏好的信息，从而在多次交互中提供个性化体验。
4. **闲聊工具**：
   - 允许代理进行闲聊。

## 环境设置

在使用此模板之前，您需要设置 Ollama 和 Neo4j 数据库。

1. 按照 [这里](https://python.langchain.com/docs/integrations/chat/ollama) 的说明下载 Ollama。

2. 下载您感兴趣的 LLM：

    * 此软件包使用 `mixtral`: `ollama pull mixtral`
    * 您可以从 [这里](https://ollama.ai/library) 选择许多 LLM

您需要定义以下环境变量

```
OLLAMA_BASE_URL=<YOUR_OLLAMA_URL>
NEO4J_URI=<YOUR_NEO4J_URI>
NEO4J_USERNAME=<YOUR_NEO4J_USERNAME>
NEO4J_PASSWORD=<YOUR_NEO4J_PASSWORD>
```

通常对于本地 Ollama 安装：

```shell
export OLLAMA_BASE_URL="http://127.0.0.1:11434"
```

## 用数据填充

如果您想用示例电影数据集填充数据库，可以运行 `python ingest.py`。该脚本导入关于电影及其用户评分的信息。此外，脚本还创建了两个 [全文索引](https://neo4j.com/docs/cypher-manual/current/indexes-for-full-text-search/)，用于将用户输入的信息映射到数据库中。

作为替代，您可以使用演示的 neo4j 推荐数据库：
```shell
export NEO4J_URI="neo4j+s://demo.neo4jlabs.com"
export NEO4J_USERNAME="recommendations"
export NEO4J_PASSWORD="recommendations"
export NEO4J_DATABASE="recommendations"
```

## 用法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的 LangChain 项目并将其作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package neo4j-semantic-ollama
```

如果您想将其添加到现有项目中，可以直接运行：

```shell
langchain app add neo4j-semantic-ollama
```

然后，在项目内，将以下代码添加到您的 `app/server.py` 文件中，替换 `add_routes(app, NotImplemented)` 部分：
```python
from neo4j_semantic_ollama import agent_executor as neo4j_semantic_agent

add_routes(app, neo4j_semantic_agent, path="/neo4j-semantic-ollama")
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

如果您在顶级项目目录中，则可以直接通过以下命令启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行于 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/neo4j-semantic-ollama/playground](http://127.0.0.1:8000/neo4j-semantic-ollama/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/neo4j-semantic-ollama")
```