# rag-astradb

此模板将使用Astra DB（`AstraDB` 向量存储类）执行RAG

## 环境设置

需要一个 [Astra DB](https://astra.datastax.com) 数据库；免费套餐即可。

- 你需要数据库的 **API 端点**（例如 `https://0123...-us-east1.apps.astra.datastax.com`）...
- ... 和一个 **令牌**（`AstraCS:...`）。

此外，还需要一个 **OpenAI API 密钥**。_请注意，默认情况下此演示仅支持 OpenAI，除非你对代码进行修改。_

通过环境变量提供连接参数和密钥。请参考 `.env.template` 以获取变量名称。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的 LangChain 项目并将此作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package rag-astradb
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-astradb
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from astradb_entomology_rag import chain as astradb_entomology_rag_chain

add_routes(app, astradb_entomology_rag_chain, path="/rag-astradb")
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

这将启动 FastAPI 应用程序，服务器在本地运行于 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/rag-astradb/playground](http://127.0.0.1:8000/rag-astradb/playground) 访问游乐场

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-astradb")
```

## 参考

独立的代码库与 LangServe 链接： [这里](https://github.com/hemidactylus/langserve_astradb_entomology_rag)。