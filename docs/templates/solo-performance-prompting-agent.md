# solo-performance-prompting-agent

此模板创建一个代理，通过与多个角色进行多轮自我协作，将单个 LLM 转变为认知协同者。认知协同者是指一个智能代理，与多个思维协作，结合它们各自的优势和知识，以增强在复杂任务中的问题解决能力和整体表现。通过根据任务输入动态识别和模拟不同角色，SPP 发挥了 LLM 中认知协同的潜力。

此模板将使用 `DuckDuckGo` 搜索 API。

## 环境设置

此模板默认使用 `OpenAI`。 
请确保在您的环境中设置了 `OPENAI_API_KEY`。

## 用法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package solo-performance-prompting-agent
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add solo-performance-prompting-agent
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from solo_performance_prompting_agent.agent import agent_executor as solo_performance_prompting_agent_chain

add_routes(app, solo_performance_prompting_agent_chain, path="/solo-performance-prompting-agent")
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

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/solo-performance-prompting-agent/playground](http://127.0.0.1:8000/solo-performance-prompting-agent/playground) 访问游乐场。

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/solo-performance-prompting-agent")
```