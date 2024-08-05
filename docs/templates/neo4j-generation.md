# neo4j-generation

此模板将基于LLM的知识图谱提取与Neo4j AuraDB配对，后者是一个完全托管的云图数据库。

您可以在[Neo4j Aura](https://neo4j.com/cloud/platform/aura-graph-database?utm_source=langchain&utm_content=langserve)上创建一个免费的实例。

当您启动一个免费的数据库实例时，您将收到访问数据库的凭据。

此模板灵活，允许用户通过指定节点标签和关系类型的列表来指导提取过程。

有关此包的功能和能力的更多详细信息，请参阅[这篇博客文章](https://blog.langchain.dev/constructing-knowledge-graphs-from-text-using-openai-functions/)。

## 环境设置

您需要设置以下环境变量：

```
OPENAI_API_KEY=<YOUR_OPENAI_API_KEY>
NEO4J_URI=<YOUR_NEO4J_URI>
NEO4J_USERNAME=<YOUR_NEO4J_USERNAME>
NEO4J_PASSWORD=<YOUR_NEO4J_PASSWORD>
```

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此包作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package neo4j-generation
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add neo4j-generation
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from neo4j_generation.chain import chain as neo4j_generation_chain

add_routes(app, neo4j_generation_chain, path="/neo4j-generation")
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

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/neo4j-generation/playground](http://127.0.0.1:8000/neo4j-generation/playground) 访问播放区。

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/neo4j-generation")
```