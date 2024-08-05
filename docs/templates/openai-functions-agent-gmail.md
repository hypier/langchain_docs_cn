# OpenAI Functions Agent - Gmail

是否曾经为达到收件箱零而苦恼？

使用此模板，您可以创建和定制您自己的 AI 助手来管理您的 Gmail 帐户。借助默认的 Gmail 工具，它可以读取、搜索和草拟电子邮件，以代表您进行回复。它还可以访问 Tavily 搜索引擎，因此在撰写之前可以搜索电子邮件线程中任何主题或人物的相关信息，确保草稿包含所有必要的信息，以显得信息丰富。

## 详细信息

该助手使用 OpenAI 的 [函数调用](https://python.langchain.com/docs/modules/chains/how_to/openai_functions) 支持，可靠地选择并调用您提供的工具。

此模板还直接从 [langchain-core](https://pypi.org/project/langchain-core/) 和 [`langchain-community`](https://pypi.org/project/langchain-community/) 导入，适当时使用。我们已重新构建 LangChain，以便您可以选择满足您用例的特定集成。虽然您仍然可以从 `langchain` 导入（我们正在使此过渡向后兼容），但我们已将大多数类的归属分开，以反映所有权并使您的依赖列表更轻。您所需的大多数集成可以在 `langchain-community` 包中找到，如果您仅使用核心表达语言 API，您甚至可以仅基于 `langchain-core` 构建。

## 环境设置

需要设置以下环境变量：

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI 模型。

设置 `TAVILY_API_KEY` 环境变量以访问 Tavily 搜索。

创建一个 [`credentials.json`](https://developers.google.com/gmail/api/quickstart/python#authorize_credentials_for_a_desktop_application) 文件，包含您来自 Gmail 的 OAuth 客户端 ID。要自定义身份验证，请参见下面的 [Customize Auth](#customize-auth) 部分。

_*注意:* 第一次运行此应用程序时，它将强制您经历用户身份验证流程。_

（可选）：将 `GMAIL_AGENT_ENABLE_SEND` 设置为 `true`（或修改此模板中的 `agent.py` 文件），以允许其访问“发送”工具。这将使您的助手在没有您明确审核的情况下发送电子邮件，这不推荐。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package openai-functions-agent-gmail
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add openai-functions-agent-gmail
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from openai_functions_agent import agent_executor as openai_functions_agent_chain

add_routes(app, openai_functions_agent_chain, path="/openai-functions-agent-gmail")
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
我们可以在 [http://127.0.0.1:8000/openai-functions-agent-gmail/playground](http://127.0.0.1:8000/openai-functions-agent/playground) 访问游乐场  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/openai-functions-agent-gmail")
```

## 自定义认证

```
from langchain_community.tools.gmail.utils import build_resource_service, get_gmail_credentials

# Can review scopes here https://developers.google.com/gmail/api/auth/scopes
# For instance, readonly scope is 'https://www.googleapis.com/auth/gmail.readonly'
credentials = get_gmail_credentials(
    token_file="token.json",
    scopes=["https://mail.google.com/"],
    client_secrets_file="credentials.json",
)
api_resource = build_resource_service(credentials=credentials)
toolkit = GmailToolkit(api_resource=api_resource)
```