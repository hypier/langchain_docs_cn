# rag-timescale-conversation

此模板用于 [会话式](https://python.langchain.com/docs/expression_language/cookbook/retrieval#conversational-retrieval-chain) [检索](https://python.langchain.com/docs/use_cases/question_answering/)，这是最受欢迎的 LLM 用例之一。

它将会话历史和检索到的文档传递给 LLM 进行综合。

## 环境设置

此模板使用 Timescale Vector 作为向量存储，并要求设置 `TIMESCALES_SERVICE_URL`。如果您还没有帐户，请在 [这里](https://console.cloud.timescale.com/signup?utm_campaign=vectorlaunch&utm_source=langchain&utm_medium=referral) 注册 90 天的试用。

要加载示例数据集，请设置 `LOAD_SAMPLE_DATA=1`。要加载您自己的数据集，请参见下面的部分。

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的 LangChain 项目并将此包作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package rag-timescale-conversation
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-timescale-conversation
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_timescale_conversation import chain as rag_timescale_conversation_chain

add_routes(app, rag_timescale_conversation_chain, path="/rag-timescale_conversation")
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
我们可以在 [http://127.0.0.1:8000/rag-timescale-conversation/playground](http://127.0.0.1:8000/rag-timescale-conversation/playground) 访问游乐场

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-timescale-conversation")
```

请参阅 `rag_conversation.ipynb` 笔记本以获取示例用法。

## 加载您自己的数据集

要加载您自己的数据集，您需要创建一个 `load_dataset` 函数。您可以在 `load_sample_dataset.py` 文件中查看 `load_ts_git_dataset` 函数的示例。然后，您可以将其作为独立函数运行（例如，在 bash 脚本中），或将其添加到 chain.py 中（但此时您只应运行一次）。