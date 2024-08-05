# 带有多个索引的RAG（路由）

一个QA应用程序，根据用户问题在不同领域特定的检索器之间进行路由。

## 环境设置

此应用程序查询 PubMed、ArXiv、Wikipedia 和 [Kay AI](https://www.kay.ai)（用于 SEC 文件）。

您需要创建一个免费的 Kay AI 账户，并在此处 [获取您的 API 密钥](https://www.kay.ai)。 
然后设置环境变量：

```bash
export KAY_API_KEY="<YOUR_API_KEY>"
```

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package rag-multi-index-router
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-multi-index-router
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_multi_index_router import chain as rag_multi_index_router_chain

add_routes(app, rag_multi_index_router_chain, path="/rag-multi-index-router")
```

（可选）现在让我们配置 LangSmith。 
LangSmith 将帮助我们追踪、监控和调试 LangChain 应用程序。 
您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。 
如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接启动一个 LangServe 实例：

```shell
langchain serve
```

这将启动一个 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/rag-multi-index-router/playground](http://127.0.0.1:8000/rag-multi-index-router/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-multi-index-router")
```