# pii-protected-chatbot

此模板创建了一个聊天机器人，它会标记任何传入的个人身份信息（PII），并且不会将其传递给LLM。

## 环境设置

需要设置以下环境变量：

将 `OPENAI_API_KEY` 环境变量设置为访问 OpenAI 模型。

## 使用方法

要使用此软件包，您首先需要安装 LangChain CLI：

```shell
pip install -U "langchain-cli[serve]"
```

要创建一个新的 LangChain 项目并将此软件包作为唯一的软件包安装，您可以执行：

```shell
langchain app new my-app --package pii-protected-chatbot
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add pii-protected-chatbot
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from pii_protected_chatbot.chain import chain as pii_protected_chatbot

add_routes(app, pii_protected_chatbot, path="/openai-functions-agent")
```

（可选）现在让我们配置 LangSmith。 
LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。 
您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。 
如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接通过以下方式启动一个 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/pii_protected_chatbot/playground](http://127.0.0.1:8000/pii_protected_chatbot/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/pii_protected_chatbot")
```