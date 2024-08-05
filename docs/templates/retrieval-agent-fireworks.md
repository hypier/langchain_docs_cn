# retrieval-agent-fireworks

该软件包使用托管在FireworksAI上的开源模型，通过代理架构进行检索。默认情况下，它对Arxiv进行检索。

我们将使用`Mixtral8x7b-instruct-v0.1`，该模型在这篇博客中显示出即使没有针对该任务进行微调，仍能产生合理的结果：https://huggingface.co/blog/open-source-llms-as-agents

## 环境设置

有多种优秀的方法来运行OSS模型。我们将使用FireworksAI作为运行模型的简单方法。有关更多信息，请参见 [这里](https://python.langchain.com/docs/integrations/providers/fireworks)。

设置 `FIREWORKS_API_KEY` 环境变量以访问Fireworks。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package retrieval-agent-fireworks
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add retrieval-agent-fireworks
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from retrieval_agent_fireworks import chain as retrieval_agent_fireworks_chain

add_routes(app, retrieval_agent_fireworks_chain, path="/retrieval-agent-fireworks")
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

如果您在此目录内，则可以直接通过以下方式启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/retrieval-agent-fireworks/playground](http://127.0.0.1:8000/retrieval-agent-fireworks/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/retrieval-agent-fireworks")
```