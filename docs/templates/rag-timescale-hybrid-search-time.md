# 使用混合搜索的时间序列向量 RAG

此模板展示了如何使用时间序列向量与自查询检索器结合，在相似性和时间上执行混合搜索。当您的数据具有强烈的时间基础组件时，这非常有用。一些此类数据的示例包括：
- 新闻文章（政治、商业等）
- 博客文章、文档或其他发布材料（公共或私人）
- 社交媒体帖子
- 任何类型的变更日志
- 消息

此类项目通常通过相似性和时间进行搜索。例如：给我展示所有关于2022年丰田卡车的新闻。

[Timescale Vector](https://www.timescale.com/ai?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 在特定时间范围内搜索嵌入时提供了卓越的性能，通过利用自动表分区来隔离特定时间范围的数据。

Langchain 的自查询检索器允许从用户查询的文本中推导时间范围（以及其他搜索标准）。

## 什么是 Timescale Vector？
**[Timescale Vector](https://www.timescale.com/ai?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 是用于 AI 应用的 PostgreSQL++。**

Timescale Vector 使您能够高效地在 `PostgreSQL` 中存储和查询数十亿个向量嵌入。
- 通过受 DiskANN 启发的索引算法，增强 `pgvector` 在 1B+ 向量上的更快、更准确的相似性搜索。
- 通过自动时间分区和索引，支持快速的基于时间的向量搜索。
- 提供熟悉的 SQL 接口，用于查询向量嵌入和关系数据。

Timescale Vector 是云 PostgreSQL，用于 AI，能够随着您从 POC 到生产的需求而扩展：
- 通过使您能够在一个数据库中存储关系元数据、向量嵌入和时间序列数据，简化操作。
- 受益于坚如磐石的 PostgreSQL 基础，具备企业级功能，如流式备份和复制、高可用性和行级安全性。
- 提供无忧体验，具备企业级安全性和合规性。

### 如何访问 Timescale Vector
Timescale Vector 可在 [Timescale](https://www.timescale.com/products?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 云 PostgreSQL 平台上使用。（目前没有自托管版本。）

- LangChain 用户可以获得 Timescale Vector 的 90 天免费试用。
- 要开始使用，请 [注册](https://console.cloud.timescale.com/signup?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) Timescale，创建一个新数据库并按照此笔记本操作！
- 有关在 Python 中使用 Timescale Vector 的更多详细信息，请参阅 [安装说明](https://github.com/timescale/python-vector)。

## 环境设置

此模板使用 Timescale Vector 作为向量存储，并要求设置 `TIMESCALES_SERVICE_URL`。如果您还没有账户，请在 [这里](https://console.cloud.timescale.com/signup?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 注册 90 天的试用。

要加载示例数据集，请设置 `LOAD_SAMPLE_DATA=1`。要加载您自己的数据集，请参见下面的部分。

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package rag-timescale-hybrid-search-time
```

如果您想将其添加到现有项目中，可以直接运行：

```shell
langchain app add rag-timescale-hybrid-search-time
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_timescale_hybrid_search.chain import chain as rag_timescale_hybrid_search_chain

add_routes(app, rag_timescale_hybrid_search_chain, path="/rag-timescale-hybrid-search")
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
我们可以在 [http://127.0.0.1:8000/rag-timescale-hybrid-search/playground](http://127.0.0.1:8000/rag-timescale-hybrid-search/playground) 访问游乐场

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-timescale-hybrid-search")
```

## 加载您自己的数据集

要加载您自己的数据集，您需要修改 `chain.py` 中的 `DATASET SPECIFIC CODE` 部分的代码。该代码定义了集合的名称、如何加载数据，以及集合内容和所有元数据的人类语言描述。人类语言描述被自查询检索器用于帮助 LLM 将问题转换为元数据的过滤器，以便在 Timescale-vector 中搜索数据。