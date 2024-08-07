# rag-vectara

该模板使用 vectara 执行 RAG。

## 环境设置

同时，确保以下环境变量已设置：
* `VECTARA_CUSTOMER_ID`
* `VECTARA_CORPUS_ID`
* `VECTARA_API_KEY`

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package rag-vectara
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-vectara
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_vectara import chain as rag_vectara_chain

add_routes(app, rag_vectara_chain, path="/rag-vectara")
```

（可选）现在让我们配置 LangSmith。 
LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。 
您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。 
如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "vectara-demo"
```

如果您在此目录中，则可以直接通过以下命令启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/rag-vectara/playground](http://127.0.0.1:8000/rag-vectara/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-vectara")
```