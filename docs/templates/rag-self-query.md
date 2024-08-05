# rag-self-query

该模板使用自查询检索技术执行RAG。主要思想是让LLM将非结构化查询转换为结构化查询。有关此工作原理的更多信息，请参见[文档](https://python.langchain.com/docs/modules/data_connection/retrievers/self_query)。

## 环境设置

在此模板中，我们将使用 OpenAI 模型和 Elasticsearch 向量存储，但该方法适用于所有 LLMs/ChatModels 以及 [多个向量存储](https://python.langchain.com/docs/integrations/retrievers/self_query/)。

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

要连接到您的 Elasticsearch 实例，请使用以下环境变量：

```bash
export ELASTIC_CLOUD_ID = <ClOUD_ID>
export ELASTIC_USERNAME = <ClOUD_USERNAME>
export ELASTIC_PASSWORD = <ClOUD_PASSWORD>
```
对于使用 Docker 的本地开发，请使用：

```bash
export ES_URL = "http://localhost:9200"
docker run -p 9200:9200 -e "discovery.type=single-node" -e "xpack.security.enabled=false" -e "xpack.security.http.ssl.enabled=false" docker.elastic.co/elasticsearch/elasticsearch:8.9.0
```

## 用法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的 LangChain 项目并将此包作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package rag-self-query
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-self-query
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_self_query import chain

add_routes(app, chain, path="/rag-elasticsearch")
```

要用示例数据填充向量存储，从目录的根目录运行：
```bash
python ingest.py
```

（可选）现在让我们配置 LangSmith。 
LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。 
您可以在此处注册 LangSmith [here](https://smith.langchain.com/)。 
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

这将启动 FastAPI 应用程序，服务器在本地运行于 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/rag-elasticsearch/playground](http://127.0.0.1:8000/rag-elasticsearch/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-self-query")
```