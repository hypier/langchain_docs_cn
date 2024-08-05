---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/connery.ipynb
---

# Connery 工具包

使用此工具包，您可以将 Connery Actions 集成到您的 LangChain 代理中。

如果您只想在代理中使用特定的 Connery Action，请查看 [Connery Action Tool](/docs/integrations/tools/connery) 文档。

## 什么是 Connery？

Connery 是一个用于 AI 的开源插件基础设施。

使用 Connery，您可以轻松创建具有一组操作的自定义插件，并将其无缝集成到您的 LangChain 代理中。Connery 将处理关键方面，如运行时、授权、秘密管理、访问管理、审计日志和其他重要功能。

此外，Connery 在我们社区的支持下，提供了一系列现成的开源插件，以便于使用。

了解有关 Connery 的更多信息：

- GitHub: https://github.com/connery-io/connery
- Documentation: https://docs.connery.io

## 前提条件

要在您的 LangChain 代理中使用 Connery Actions，您需要进行一些准备：

1. 使用 [快速入门](https://docs.connery.io/docs/runner/quick-start/) 指南设置 Connery 运行器。
2. 安装您希望在代理中使用的所有插件和操作。
3. 设置环境变量 `CONNERY_RUNNER_URL` 和 `CONNERY_RUNNER_API_KEY`，以便工具包能够与 Connery 运行器进行通信。

## 使用 Connery Toolkit 的示例

在下面的示例中，我们创建了一个代理，使用两个 Connery 动作来总结一个公共网页并通过电子邮件发送摘要：

1. 来自 [Summarization](https://github.com/connery-io/summarization-plugin) 插件的 **总结公共网页** 动作。
2. 来自 [Gmail](https://github.com/connery-io/gmail) 插件的 **发送电子邮件** 动作。

您可以在 [这里](https://smith.langchain.com/public/4af5385a-afe9-46f6-8a53-57fe2d63c5bc/r) 查看此示例的 LangSmith 跟踪。

```python
%pip install -qU langchain-community
```

```python
import os

from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits.connery import ConneryToolkit
from langchain_community.tools.connery import ConneryService
from langchain_openai import ChatOpenAI

# Specify your Connery Runner credentials.
os.environ["CONNERY_RUNNER_URL"] = ""
os.environ["CONNERY_RUNNER_API_KEY"] = ""

# Specify OpenAI API key.
os.environ["OPENAI_API_KEY"] = ""

# Specify your email address to receive the email with the summary from example below.
recepient_email = "test@example.com"

# Create a Connery Toolkit with all the available actions from the Connery Runner.
connery_service = ConneryService()
connery_toolkit = ConneryToolkit.create_instance(connery_service)

# Use OpenAI Functions agent to execute the prompt using actions from the Connery Toolkit.
llm = ChatOpenAI(temperature=0)
agent = initialize_agent(
    connery_toolkit.get_tools(), llm, AgentType.OPENAI_FUNCTIONS, verbose=True
)
result = agent.run(
    f"""Make a short summary of the webpage http://www.paulgraham.com/vb.html in three sentences
and send it to {recepient_email}. Include the link to the webpage into the body of the email."""
)
print(result)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Invoking: `CA72DFB0AB4DF6C830B43E14B0782F70` with `{'publicWebpageUrl': 'http://www.paulgraham.com/vb.html'}`


[0m[33;1m[1;3m{'summary': '作者反思了生命短暂的概念，以及拥有孩子让他们意识到生命的真正短暂。他们讨论了时间如何可以转化为离散的量，以及某些经历是多么有限。作者强调了在生活中优先考虑和消除不必要事物的重要性，以及积极追求有意义的经历。他们还讨论了卷入网络争论的负面影响，以及意识到时间如何被使用的必要性。作者建议修剪不必要的活动，不要等待去做重要的事情，并珍惜自己拥有的时间。'}[0m[32;1m[1;3m
Invoking: `CABC80BB79C15067CA983495324AE709` with `{'recipient': 'test@example.com', 'subject': 'Summary of the webpage', 'body': 'Here is a short summary of the webpage http://www.paulgraham.com/vb.html:\n\n作者反思了生命短暂的概念，以及拥有孩子让他们意识到生命的真正短暂。他们讨论了时间如何可以转化为离散的量，以及某些经历是多么有限。作者强调了在生活中优先考虑和消除不必要事物的重要性，以及积极追求有意义的经历。他们还讨论了卷入网络争论的负面影响，以及意识到时间如何被使用的必要性。作者建议修剪不必要的活动，不要等待去做重要的事情，并珍惜自己拥有的时间。\n\n您可以在此处找到完整网页 [here](http://www.paulgraham.com/vb.html).'}`


[0m[33;1m[1;3m{'messageId': '<2f04b00e-122d-c7de-c91e-e78e0c3276d6@gmail.com>'}[0m[32;1m[1;3m我已将网页摘要的电子邮件发送至 test@example.com。请检查您的收件箱。[0m

[1m> Finished chain.[0m
我已将网页摘要的电子邮件发送至 test@example.com。请检查您的收件箱。
```
注意：Connery 动作是结构化工具，因此您只能在支持结构化工具的代理中使用它。