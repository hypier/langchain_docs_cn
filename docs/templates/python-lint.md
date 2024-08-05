# python-lint

该代理专注于生成高质量的 Python 代码，重点关注正确的格式和代码检查。它使用 `black`、`ruff` 和 `mypy` 来确保代码符合标准质量检查。

这通过集成和响应这些检查来简化编码过程，从而生成可靠且一致的代码输出。

它实际上无法执行所写的代码，因为代码执行可能引入额外的依赖项和潜在的安全漏洞。这使得该代理成为一个安全且高效的代码生成解决方案。

您可以直接使用它生成 Python 代码，或将其与规划和执行代理网络连接。

## 环境设置

- 安装 `black`、`ruff` 和 `mypy`：`pip install -U black ruff mypy`
- 设置 `OPENAI_API_KEY` 环境变量。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将此作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package python-lint
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add python-lint
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from python_lint import agent_executor as python_lint_agent

add_routes(app, python_lint_agent, path="/python-lint")
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
我们可以在 [http://127.0.0.1:8000/python-lint/playground](http://127.0.0.1:8000/python-lint/playground) 访问游乐场  

我们可以通过以下代码从代码中访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/python-lint")
```