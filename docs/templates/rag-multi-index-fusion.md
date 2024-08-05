# RAG与多个索引（融合）

一个查询多个特定领域检索器的QA应用程序，并从所有检索结果中选择最相关的文档。

## 环境设置

此应用程序查询 PubMed、ArXiv、Wikipedia 和 [Kay AI](https://www.kay.ai)（用于 SEC 文件）。

您需要创建一个免费的 Kay AI 账户，并 [在这里获取您的 API 密钥](https://www.kay.ai)。  
然后设置环境变量：

```bash
export KAY_API_KEY="<YOUR_API_KEY>"
```

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package rag-multi-index-fusion
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-multi-index-fusion
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_multi_index_fusion import chain as rag_multi_index_fusion_chain

add_routes(app, rag_multi_index_fusion_chain, path="/rag-multi-index-fusion")
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

这将启动 FastAPI 应用程序，服务器在本地运行于 
[http://localhost:8000](http://localhost:8000)。

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/rag-multi-index-fusion/playground](http://127.0.0.1:8000/rag-multi-index-fusion/playground) 访问游乐场。

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-multi-index-fusion")
```