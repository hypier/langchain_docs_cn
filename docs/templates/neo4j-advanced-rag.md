# neo4j-advanced-rag

此模板通过实施高级检索策略，使您能够平衡精确的嵌入和上下文保留。

## 策略

1. **典型的 RAG**：
   - 传统方法，其中索引的确切数据就是检索到的数据。
2. **父检索器**：
   - 数据不是索引整个文档，而是被分成更小的块，称为父文档和子文档。
   - 子文档被索引以更好地表示特定概念，而父文档则被检索以确保上下文的保留。
3. **假设性问题**：
     - 文档经过处理以确定它们可能回答的潜在问题。
     - 这些问题随后被索引以更好地表示特定概念，而父文档则被检索以确保上下文的保留。
4. **摘要**：
     - 不是索引整个文档，而是创建文档的摘要并进行索引。
     - 类似地，在 RAG 应用中检索父文档。

## 环境设置

您需要定义以下环境变量

```
OPENAI_API_KEY=<YOUR_OPENAI_API_KEY>
NEO4J_URI=<YOUR_NEO4J_URI>
NEO4J_USERNAME=<YOUR_NEO4J_USERNAME>
NEO4J_PASSWORD=<YOUR_NEO4J_PASSWORD>
```

## 用数据填充

如果您想用一些示例数据填充数据库，可以运行 `python ingest.py`。该脚本处理并将文件 `dune.txt` 中的文本部分存储到 Neo4j 图形数据库中。首先，文本被划分为较大的块（“父块”），然后进一步细分为较小的块（“子块”），其中父块和子块之间有轻微重叠，以保持上下文。在将这些块存储到数据库后，使用 OpenAI 的 embeddings 计算子节点的嵌入，并将其存回图中以便于未来的检索或分析。对于每个父节点，生成假设性问题和摘要，进行嵌入，并添加到数据库中。此外，为每种检索策略创建一个向量索引，以高效查询这些嵌入。

*请注意，由于 LLM 生成假设性问题和摘要的速度，数据摄取可能需要一到两分钟。*

## 使用方法

要使用此软件包，您首先需要安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的 LangChain 项目并将其作为唯一的软件包安装，您可以执行：

```shell
langchain app new my-app --package neo4j-advanced-rag
```

如果您想将其添加到现有项目中，可以直接运行：

```shell
langchain app add neo4j-advanced-rag
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from neo4j_advanced_rag import chain as neo4j_advanced_chain

add_routes(app, neo4j_advanced_chain, path="/neo4j-advanced-rag")
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
我们可以在 [http://127.0.0.1:8000/neo4j-advanced-rag/playground](http://127.0.0.1:8000/neo4j-advanced-rag/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/neo4j-advanced-rag")
```