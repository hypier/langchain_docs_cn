# Weaviate中的混合搜索
此模板展示了如何使用Weaviate中的混合搜索功能。混合搜索结合了多种搜索算法，以提高搜索结果的准确性和相关性。

Weaviate使用稀疏和密集向量来表示搜索查询和文档的含义和上下文。结果使用`bm25`和向量搜索排名的组合来返回最佳结果。

## 配置
通过在 `chain.py` 中设置一些环境变量连接到您的托管 Weaviate 向量存储：

* `WEAVIATE_ENVIRONMENT`
* `WEAVIATE_API_KEY`

您还需要设置您的 `OPENAI_API_KEY` 以使用 OpenAI 模型。

## 开始使用  
要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package hybrid-search-weaviate
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add hybrid-search-weaviate
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from hybrid_search_weaviate import chain as hybrid_search_weaviate_chain

add_routes(app, hybrid_search_weaviate_chain, path="/hybrid-search-weaviate")
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
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板  
我们可以在 [http://127.0.0.1:8000/hybrid-search-weaviate/playground](http://127.0.0.1:8000/hybrid-search-weaviate/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/hybrid-search-weaviate")
```