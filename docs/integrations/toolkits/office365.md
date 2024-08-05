---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/office365.ipynb
---

# Office365

>[Microsoft 365](https://www.office.com/) 是由 `Microsoft` 拥有的一系列生产力软件、协作和基于云的服务产品。
>
>注意：`Office 365` 已重新品牌为 `Microsoft 365`。

本笔记本介绍如何将 LangChain 连接到 `Office365` 电子邮件和日历。

要使用此工具包，您需要设置凭据，详细信息请参见 [Microsoft Graph 身份验证和授权概述](https://learn.microsoft.com/en-us/graph/auth/)。一旦您获得了 CLIENT_ID 和 CLIENT_SECRET，您可以将它们作为环境变量输入到下面。

您还可以使用 [此处的身份验证说明](https://o365.github.io/python-o365/latest/getting_started.html#oauth-setup-pre-requisite)。

```python
%pip install --upgrade --quiet  O365
%pip install --upgrade --quiet  beautifulsoup4  # This is optional but is useful for parsing HTML messages
%pip install -qU langchain-community
```

## 设置环境变量

工具包将读取 `CLIENT_ID` 和 `CLIENT_SECRET` 环境变量以进行用户身份验证，因此您需要在此处设置它们。您还需要设置您的 `OPENAI_API_KEY` 以便稍后使用该代理。

```python
# Set environmental variables here
```

## 创建工具包并获取工具

首先，您需要创建工具包，以便稍后可以访问其工具。

```python
from langchain_community.agent_toolkits import O365Toolkit

toolkit = O365Toolkit()
tools = toolkit.get_tools()
tools
```

```output
[O365SearchEvents(name='events_search', description=" 使用此工具搜索用户的日历事件。输入必须是搜索查询的开始和结束日期时间。输出是用户日历中在开始和结束时间之间的所有事件的 JSON 列表。您可以假设用户在已有会议期间无法安排任何会议，并且用户在会议期间是忙碌的。没有事件的时间对用户是空闲的。", args_schema=<class 'langchain_community.tools.office365.events_search.SearchEventsInput'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, handle_tool_error=False, account=Account Client Id: f32a022c-3c4c-4d10-a9d8-f6a9a9055302),
 O365CreateDraftMessage(name='create_email_draft', description='使用此工具创建带有提供的消息字段的草稿电子邮件。', args_schema=<class 'langchain_community.tools.office365.create_draft_message.CreateDraftMessageSchema'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, handle_tool_error=False, account=Account Client Id: f32a022c-3c4c-4d10-a9d8-f6a9a9055302),
 O365SearchEmails(name='messages_search', description='使用此工具搜索电子邮件消息。输入必须是有效的 Microsoft Graph v1.0 $search 查询。输出是请求资源的 JSON 列表。', args_schema=<class 'langchain_community.tools.office365.messages_search.SearchEmailsInput'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, handle_tool_error=False, account=Account Client Id: f32a022c-3c4c-4d10-a9d8-f6a9a9055302),
 O365SendEvent(name='send_event', description='使用此工具创建并发送带有提供的事件字段的事件。', args_schema=<class 'langchain_community.tools.office365.send_event.SendEventSchema'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, handle_tool_error=False, account=Account Client Id: f32a022c-3c4c-4d10-a9d8-f6a9a9055302),
 O365SendMessage(name='send_email', description='使用此工具发送带有提供的消息字段的电子邮件。', args_schema=<class 'langchain_community.tools.office365.send_message.SendMessageSchema'>, return_direct=False, verbose=False, callbacks=None, callback_manager=None, handle_tool_error=False, account=Account Client Id: f32a022c-3c4c-4d10-a9d8-f6a9a9055302)]
```

## 在代理中使用


```python
from langchain.agents import AgentType, initialize_agent
from langchain_openai import OpenAI
```


```python
llm = OpenAI(temperature=0)
agent = initialize_agent(
    tools=toolkit.get_tools(),
    llm=llm,
    verbose=False,
    agent=AgentType.STRUCTURED_CHAT_ZERO_SHOT_REACT_DESCRIPTION,
)
```


```python
agent.run(
    "为我创建一封电子邮件草稿，内容是一个有意识的鹦鹉"
    " 希望与她的"
    " 疏远朋友，一只猫，合作进行一些研究。无论如何，你都不能发送这条消息。"
)
```



```output
'草稿电子邮件已正确创建。'
```



```python
agent.run(
    "你能在我的草稿文件夹中搜索一下，看看是否有关于合作的草稿吗？"
)
```



```output
"我在你的草稿文件夹中找到了一个关于合作的草稿。它是在2023-06-16T18:22:17+0000发送的，主题是'合作请求'。"
```



```python
agent.run(
    "你能在2023年10月3日下午2点（东部时间）安排与一只有意识的鹦鹉进行30分钟的会议，讨论研究合作吗？"
)
```
```output
/home/vscode/langchain-py-env/lib/python3.11/site-packages/O365/utils/windows_tz.py:639: PytzUsageWarning: The zone attribute is specific to pytz's interface; please migrate to a new time zone provider. For more details on how to do so, see https://pytz-deprecation-shim.readthedocs.io/en/latest/migration.html
  iana_tz.zone if isinstance(iana_tz, tzinfo) else iana_tz)
/home/vscode/langchain-py-env/lib/python3.11/site-packages/O365/utils/utils.py:463: PytzUsageWarning: The zone attribute is specific to pytz's interface; please migrate to a new time zone provider. For more details on how to do so, see https://pytz-deprecation-shim.readthedocs.io/en/latest/migration.html
  timezone = date_time.tzinfo.zone if date_time.tzinfo is not None else None
```


```output
'我已安排与一只有意识的鹦鹉在2023年10月3日下午2点讨论研究合作的会议。如果需要做任何更改，请告诉我。'
```



```python
agent.run(
    "你能告诉我2023年10月3日东部时间是否有任何事件吗？如果有，请告诉我是否有与一只有意识的鹦鹉的事件？"
)
```



```output
"是的，2023年10月3日你有一个与一只有意识的鹦鹉的事件。该事件的标题是'与有意识的鹦鹉的会议'，定于下午6:00到6:30。"
```