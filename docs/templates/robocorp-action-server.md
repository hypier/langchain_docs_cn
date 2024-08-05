# Langchain - Robocorp Action Server

此模板使得可以将 [Robocorp Action Server](https://github.com/robocorp/robocorp) 提供的操作作为代理的工具。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package robocorp-action-server
```

如果您想将其添加到现有项目中，您只需运行：

```shell
langchain app add robocorp-action-server
```

并将以下代码添加到您的 `server.py` 文件中：

```python
from robocorp_action_server import agent_executor as action_server_chain

add_routes(app, action_server_chain, path="/robocorp-action-server")
```

### 运行 Action Server

要运行 Action Server，您需要安装 Robocorp Action Server

```bash
pip install -U robocorp-action-server
```

然后您可以使用以下命令运行 Action Server：

```bash
action-server new
cd ./your-project-name
action-server start
```

### 配置 LangSmith（可选）

LangSmith 将帮助我们追踪、监控和调试 LangChain 应用程序。  
您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。  
如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # if not specified, defaults to "default"
```

### 启动 LangServe 实例

如果您在此目录中，则可以通过以下方式直接启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/robocorp-action-server/playground](http://127.0.0.1:8000/robocorp-action-server/playground) 访问游乐场

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/robocorp-action-server")
```