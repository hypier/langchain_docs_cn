# elastic-query-generator

此模板允许使用 LLM 以自然语言与 Elasticsearch 分析数据库进行交互。

它通过 Elasticsearch DSL API 构建搜索查询（过滤器和聚合）。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

### 安装 Elasticsearch

有多种方式可以运行 Elasticsearch。然而，推荐的一种方式是通过 Elastic Cloud。

在 [Elastic Cloud](https://cloud.elastic.co/registration?utm_source=langchain&utm_content=langserve) 上创建一个免费试用账户。

有了部署后，更新连接字符串。

密码和连接（elasticsearch url）可以在部署控制台中找到。

请注意，Elasticsearch 客户端必须具有索引列出、映射描述和搜索查询的权限。

### 用数据填充

如果您想用一些示例信息填充数据库，可以运行 `python ingest.py`。

这将创建一个 `customers` 索引。在此包中，我们指定要生成查询的索引，并指定 `["customers"]`。这与设置您的 Elastic 索引相关。

## 使用方法

要使用此软件包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的软件包安装，您可以执行：

```shell
langchain app new my-app --package elastic-query-generator
```

如果您想将其添加到现有项目中，您只需运行：

```shell
langchain app add elastic-query-generator
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from elastic_query_generator.chain import chain as elastic_query_generator_chain

add_routes(app, elastic_query_generator_chain, path="/elastic-query-generator")
```

（可选）现在让我们配置 LangSmith。 
LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。 
您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。 
如果您没有访问权限，可以跳过此部分

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
我们可以在 [http://127.0.0.1:8000/elastic-query-generator/playground](http://127.0.0.1:8000/elastic-query-generator/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/elastic-query-generator")
```