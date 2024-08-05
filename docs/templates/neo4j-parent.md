# neo4j-parent

此模板允许您通过将文档拆分为较小的块并检索其原始或更大文本信息来平衡精确嵌入和上下文保留。

使用 Neo4j 向量索引，该包通过向量相似性搜索查询子节点，并通过定义适当的 `retrieval_query` 参数来检索相应父节点的文本。

## 环境设置

您需要定义以下环境变量

```
OPENAI_API_KEY=<YOUR_OPENAI_API_KEY>
NEO4J_URI=<YOUR_NEO4J_URI>
NEO4J_USERNAME=<YOUR_NEO4J_USERNAME>
NEO4J_PASSWORD=<YOUR_NEO4J_PASSWORD>
```

## 使用数据填充

如果您想用一些示例数据填充数据库，可以运行 `python ingest.py`。  
该脚本处理并存储来自文件 `dune.txt` 的文本部分到 Neo4j 图形数据库中。  
首先，文本被分为较大的块（“父”），然后进一步细分为较小的块（“子”），其中父块和子块之间有轻微重叠，以保持上下文。  
在将这些块存储到数据库后，使用 OpenAI 的嵌入计算子节点的嵌入，并将其存储回图中，以便将来检索或分析。  
此外，还创建了一个名为 `retrieval` 的向量索引，以便高效查询这些嵌入。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此包作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package neo4j-parent
```

如果您想将此包添加到现有项目中，只需运行：

```shell
langchain app add neo4j-parent
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from neo4j_parent import chain as neo4j_parent_chain

add_routes(app, neo4j_parent_chain, path="/neo4j-parent")
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

如果您在此目录中，则可以直接通过以下命令启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/neo4j-parent/playground](http://127.0.0.1:8000/neo4j-parent/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/neo4j-parent")
```