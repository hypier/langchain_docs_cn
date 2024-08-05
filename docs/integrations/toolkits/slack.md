---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/slack.ipynb
sidebar_label: Slack
---

# SlackToolkit

这将帮助您开始使用 Slack [工具包](/docs/concepts/#toolkits)。有关所有 SlackToolkit 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/agent_toolkits/langchain_community.agent_toolkits.slack.toolkit.SlackToolkit.html)。

## 设置

要使用此工具包，您需要获取一个令牌，具体说明请参见[Slack API 文档](https://api.slack.com/tutorials/tracks/getting-a-token)。一旦您收到 SLACK_USER_TOKEN，您可以将其作为环境变量输入如下。

```python
import getpass
import os

if not os.getenv("SLACK_USER_TOKEN"):
    os.environ["SLACK_USER_TOKEN"] = getpass.getpass("Enter your Slack user token: ")
```

如果您希望从单个工具的运行中获得自动跟踪，您还可以通过取消下面的注释来设置您的[LangSmith](https://docs.smith.langchain.com/) API 密钥：

```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### 安装

此工具包位于 `langchain-community` 包中。我们还需要 Slack SDK：


```python
%pip install -qU langchain-community slack_sdk
```

可选地，我们可以安装 beautifulsoup4 以帮助解析 HTML 消息：


```python
%pip install -qU beautifulsoup4 # 这是可选的，但对于解析 HTML 消息非常有用
```

## 实例化

现在我们可以实例化我们的工具包：


```python
from langchain_community.agent_toolkits import SlackToolkit

toolkit = SlackToolkit()
```

## 工具

查看可用工具：


```python
tools = toolkit.get_tools()

tools
```



```output
[SlackGetChannel(client=<slack_sdk.web.client.WebClient object at 0x113caa8c0>),
 SlackGetMessage(client=<slack_sdk.web.client.WebClient object at 0x113caa4d0>),
 SlackScheduleMessage(client=<slack_sdk.web.client.WebClient object at 0x113caa440>),
 SlackSendMessage(client=<slack_sdk.web.client.WebClient object at 0x113caa410>)]
```


该工具包加载了：

- [SlackGetChannel](https://api.python.langchain.com/en/latest/tools/langchain_community.tools.slack.get_channel.SlackGetChannel.html)
- [SlackGetMessage](https://api.python.langchain.com/en/latest/tools/langchain_community.tools.slack.get_message.SlackGetMessage.html)
- [SlackScheduleMessage](https://api.python.langchain.com/en/latest/tools/langchain_community.tools.slack.schedule_message.SlackScheduleMessage.html)
- [SlackSendMessage](https://api.python.langchain.com/en/latest/tools/langchain_community.tools.slack.send_message.SlackSendMessage.html)

## 在代理中使用

让我们为代理配备 Slack 工具包，并查询有关频道的信息。

```python
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import create_react_agent

llm = ChatOpenAI(model="gpt-3.5-turbo-0125")

agent_executor = create_react_agent(llm, tools)
```

```python
example_query = "When was the #general channel created?"

events = agent_executor.stream(
    {"messages": [("user", example_query)]},
    stream_mode="values",
)
for event in events:
    message = event["messages"][-1]
    if message.type != "tool":  # mask sensitive information
        event["messages"][-1].pretty_print()
```
```output
================================[1m 人类消息 [0m=================================

When was the #general channel created?
==================================[1m AI 消息 [0m==================================
工具调用:
  get_channelid_name_dict (call_NXDkALjoOx97uF1v0CoZTqtJ)
 调用 ID: call_NXDkALjoOx97uF1v0CoZTqtJ
  参数:
==================================[1m AI 消息 [0m==================================

The #general channel was created on timestamp 1671043305.
```

```python
example_query = "Send a friendly greeting to channel C072Q1LP4QM."

events = agent_executor.stream(
    {"messages": [("user", example_query)]},
    stream_mode="values",
)
for event in events:
    message = event["messages"][-1]
    if message.type != "tool":  # mask sensitive information
        event["messages"][-1].pretty_print()
```
```output
================================[1m 人类消息 [0m=================================

Send a friendly greeting to channel C072Q1LP4QM.
==================================[1m AI 消息 [0m==================================
工具调用:
  send_message (call_xQxpv4wFeAZNZgSBJRIuaizi)
 调用 ID: call_xQxpv4wFeAZNZgSBJRIuaizi
  参数:
    message: Hello! Have a great day!
    channel: C072Q1LP4QM
==================================[1m AI 消息 [0m==================================

I have sent a friendly greeting to the channel C072Q1LP4QM.
```

## API 参考

有关所有 `SlackToolkit` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/agent_toolkits/langchain_community.agent_toolkits.slack.toolkit.SlackToolkit.html)。