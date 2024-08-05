---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/connery.ipynb
---

# Connery Action Tool

使用此工具，您可以将单个 Connery Action 集成到您的 LangChain 代理中。

如果您想在代理中使用多个 Connery Action，请查看 [Connery Toolkit](/docs/integrations/toolkits/connery) 文档。

## 什么是 Connery？

Connery 是一个用于 AI 的开源插件基础设施。

使用 Connery，您可以轻松创建自定义插件，设置一系列操作，并将其无缝集成到您的 LangChain 代理中。Connery 将处理运行时、授权、秘密管理、访问管理、审计日志和其他重要功能等关键方面。

此外，Connery 在我们的社区支持下，提供了一系列多样化的现成开源插件，以便于使用。

了解更多关于 Connery 的信息：

- GitHub: https://github.com/connery-io/connery
- Documentation: https://docs.connery.io

## 前提条件

要在您的 LangChain 代理中使用 Connery Actions，您需要进行一些准备：

1. 使用 [快速入门](https://docs.connery.io/docs/runner/quick-start/) 指南设置 Connery 运行器。
2. 安装您希望在代理中使用的所有插件和操作。
3. 设置环境变量 `CONNERY_RUNNER_URL` 和 `CONNERY_RUNNER_API_KEY` 以便工具包能够与 Connery 运行器通信。

## 使用 Connery Action Tool 的示例

在下面的示例中，我们通过其 ID 从 Connery Runner 获取操作，然后使用指定的参数调用它。

在这里，我们使用来自 [Gmail](https://github.com/connery-io/gmail) 插件的 **发送邮件** 操作的 ID。

```python
%pip install -upgrade --quiet langchain-community
```

```python
import os

from langchain.agents import AgentType, initialize_agent
from langchain_community.tools.connery import ConneryService
from langchain_openai import ChatOpenAI

# Specify your Connery Runner credentials.
os.environ["CONNERY_RUNNER_URL"] = ""
os.environ["CONNERY_RUNNER_API_KEY"] = ""

# Specify OpenAI API key.
os.environ["OPENAI_API_KEY"] = ""

# Specify your email address to receive the emails from examples below.
recepient_email = "test@example.com"

# Get the SendEmail action from the Connery Runner by ID.
connery_service = ConneryService()
send_email_action = connery_service.get_action("CABC80BB79C15067CA983495324AE709")
```

手动运行该操作。

```python
manual_run_result = send_email_action.run(
    {
        "recipient": recepient_email,
        "subject": "Test email",
        "body": "This is a test email sent from Connery.",
    }
)
print(manual_run_result)
```

使用 OpenAI Functions 代理运行该操作。

您可以在此处查看此示例的 LangSmith 跟踪 [here](https://smith.langchain.com/public/a37d216f-c121-46da-a428-0e09dc19b1dc/r)。

```python
llm = ChatOpenAI(temperature=0)
agent = initialize_agent(
    [send_email_action], llm, AgentType.OPENAI_FUNCTIONS, verbose=True
)
agent_run_result = agent.run(
    f"Send an email to the {recepient_email} and say that I will be late for the meeting."
)
print(agent_run_result)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Invoking: `CABC80BB79C15067CA983495324AE709` with `{'recipient': 'test@example.com', 'subject': 'Late for Meeting', 'body': 'Dear Team,\n\nI wanted to inform you that I will be late for the meeting today. I apologize for any inconvenience caused. Please proceed with the meeting without me and I will join as soon as I can.\n\nBest regards,\n[Your Name]'}`


[0m[36;1m[1;3m{'messageId': '<d34a694d-50e0-3988-25da-e86b4c51d7a7@gmail.com>'}[0m[32;1m[1;3mI have sent an email to test@example.com informing them that you will be late for the meeting.[0m

[1m> Finished chain.[0m
I have sent an email to test@example.com informing them that you will be late for the meeting.
```
注意：Connery Action 是一个结构化工具，因此您只能在支持结构化工具的代理中使用它。

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)