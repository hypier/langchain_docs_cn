---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/zapier.ipynb
---

# Zapier 自然语言操作

**已弃用** 此 API 将于 2023-11-17 停止服务: https://nla.zapier.com/start/

>[Zapier 自然语言操作](https://nla.zapier.com/start/) 通过自然语言 API 接口为您提供对 Zapier 平台上 5000 多个应用程序和 20000 多个操作的访问权限。
>
>NLA 支持的应用程序包括 `Gmail`、`Salesforce`、`Trello`、`Slack`、`Asana`、`HubSpot`、`Google Sheets`、`Microsoft Teams` 以及其他数千个应用程序: https://zapier.com/apps
>`Zapier NLA` 处理所有底层 API 认证和从自然语言到底层 API 调用的转换，并为 LLM 返回简化输出。关键思想是您或您的用户通过类似 oauth 的设置窗口暴露一组操作，然后您可以通过 REST API 查询和执行这些操作。

NLA 提供 API 密钥和 OAuth 两种方式来签署 NLA API 请求。

1. 服务器端（API 密钥）：用于快速入门、测试和生产场景，在这些场景中，LangChain 仅使用开发者的 Zapier 账户中暴露的操作（并将使用开发者在 Zapier.com 上连接的账户）。

2. 面向用户（Oauth）：用于您正在部署面向最终用户的应用程序的生产场景，LangChain 需要访问最终用户在 Zapier.com 上暴露的操作和连接的账户。

本快速入门主要集中于服务器端用例以简洁为主。跳转到 [使用 OAuth 访问令牌的示例](#oauth) 查看如何为面向用户的情况设置 Zapier 的简短示例。查看 [完整文档](https://nla.zapier.com/start/) 以获取完整的面向用户的 oauth 开发者支持。

此示例介绍了如何使用 `SimpleSequentialChain` 和 `Agent` 的 Zapier 集成。
代码如下：


```python
import os

# get from https://platform.openai.com/
os.environ["OPENAI_API_KEY"] = os.environ.get("OPENAI_API_KEY", "")

# get from https://nla.zapier.com/docs/authentication/ after logging in):
os.environ["ZAPIER_NLA_API_KEY"] = os.environ.get("ZAPIER_NLA_API_KEY", "")
```

## 示例与代理
Zapier 工具可以与代理一起使用。请参见下面的示例。

```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits import ZapierToolkit
from langchain_community.utilities.zapier import ZapierNLAWrapper
from langchain_openai import OpenAI
```

```python
## 步骤 0. 暴露 Gmail 的“查找邮件”和 Slack 的“发送频道消息”操作

# 首先访问此处，登录，暴露（启用）这两个操作：https://nla.zapier.com/demo/start -- 对于此示例，可以将所有字段留为“让 AI 猜测”
# 在 OAuth 场景中，您将获得自己的 <provider> id（而不是 'demo'），您首先将用户引导到该 id
```

```python
llm = OpenAI(temperature=0)
zapier = ZapierNLAWrapper()
toolkit = ZapierToolkit.from_zapier_nla_wrapper(zapier)
agent = initialize_agent(
    toolkit.get_tools(), llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

```python
agent.run(
    "总结我收到的关于硅谷银行的最后一封邮件。将摘要发送到 Slack 中的 #test-zapier 频道。"
)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m I need to find the email and summarize it.
Action: Gmail: Find Email
Action Input: Find the latest email from Silicon Valley Bank[0m
Observation: [31;1m[1;3m{"from__name": "Silicon Valley Bridge Bank, N.A.", "from__email": "sreply@svb.com", "body_plain": "Dear Clients, After chaotic, tumultuous & stressful days, we have clarity on path for SVB, FDIC is fully insuring all deposits & have an ask for clients & partners as we rebuild. Tim Mayopoulos <https://eml.svb.com/NjEwLUtBSy0yNjYAAAGKgoxUeBCLAyF_NxON97X4rKEaNBLG", "reply_to__email": "sreply@svb.com", "subject": "Meet the new CEO Tim Mayopoulos", "date": "Tue, 14 Mar 2023 23:42:29 -0500 (CDT)", "message_url": "https://mail.google.com/mail/u/0/#inbox/186e393b13cfdf0a", "attachment_count": "0", "to__emails": "ankush@langchain.dev", "message_id": "186e393b13cfdf0a", "labels": "IMPORTANT, CATEGORY_UPDATES, INBOX"}[0m
Thought:[32;1m[1;3m I need to summarize the email and send it to the #test-zapier channel in Slack.
Action: Slack: Send Channel Message
Action Input: Send a slack message to the #test-zapier channel with the text "Silicon Valley Bank has announced that Tim Mayopoulos is the new CEO. FDIC is fully insuring all deposits and they have an ask for clients and partners as they rebuild."[0m
Observation: [36;1m[1;3m{"message__text": "Silicon Valley Bank has announced that Tim Mayopoulos is the new CEO. FDIC is fully insuring all deposits and they have an ask for clients and partners as they rebuild.", "message__permalink": "https://langchain.slack.com/archives/C04TSGU0RA7/p1678859932375259", "channel": "C04TSGU0RA7", "message__bot_profile__name": "Zapier", "message__team": "T04F8K3FZB5", "message__bot_id": "B04TRV4R74K", "message__bot_profile__deleted": "false", "message__bot_profile__app_id": "A024R9PQM", "ts_time": "2023-03-15T05:58:52Z", "message__bot_profile__icons__image_36": "https://avatars.slack-edge.com/2022-08-02/3888649620612_f864dc1bb794cf7d82b0_36.png", "message__blocks[]block_id": "kdZZ", "message__blocks[]elements[]type": "['rich_text_section']"}[0m
Thought:[32;1m[1;3m I now know the final answer.
Final Answer: I have sent a summary of the last email from Silicon Valley Bank to the #test-zapier channel in Slack.[0m

[1m> Finished chain.[0m
```

```output
'I have sent a summary of the last email from Silicon Valley Bank to the #test-zapier channel in Slack.'
```

## 示例：使用 SimpleSequentialChain
如果您需要更明确的控制，请使用如下链。

```python
from langchain.chains import LLMChain, SimpleSequentialChain, TransformChain
from langchain_community.tools.zapier.tool import ZapierNLARunAction
from langchain_community.utilities.zapier import ZapierNLAWrapper
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
```

```python
## 步骤 0. 暴露 Gmail 的“查找电子邮件”和 Slack 的“发送直接消息”操作

# 首先访问此处，登录，暴露（启用）这两个操作：https://nla.zapier.com/demo/start -- 对于这个示例，可以将所有字段留为“让 AI 猜测”
# 在 OAuth 场景中，您将获得自己的 <provider> id（而不是 'demo'），您需要先通过它引导用户

actions = ZapierNLAWrapper().list()
```

```python
## 步骤 1. Gmail 查找电子邮件

GMAIL_SEARCH_INSTRUCTIONS = "获取来自硅谷银行的最新电子邮件"


def nla_gmail(inputs):
    action = next(
        (a for a in actions if a["description"].startswith("Gmail: Find Email")), None
    )
    return {
        "email_data": ZapierNLARunAction(
            action_id=action["id"],
            zapier_description=action["description"],
            params_schema=action["params"],
        ).run(inputs["instructions"])
    }


gmail_chain = TransformChain(
    input_variables=["instructions"],
    output_variables=["email_data"],
    transform=nla_gmail,
)
```

```python
## 步骤 2. 生成草稿回复

template = """您是一个助手，负责草拟对 incoming email 的回复。以纯文本格式输出草稿回复（而不是 JSON）。

Incoming email:
{email_data}

Draft email reply:"""

prompt_template = PromptTemplate(input_variables=["email_data"], template=template)
reply_chain = LLMChain(llm=OpenAI(temperature=0.7), prompt=prompt_template)
```

```python
## 步骤 3. 通过 Slack 直接消息发送草稿回复

SLACK_HANDLE = "@Ankush Gola"


def nla_slack(inputs):
    action = next(
        (
            a
            for a in actions
            if a["description"].startswith("Slack: Send Direct Message")
        ),
        None,
    )
    instructions = f'将此发送给 {SLACK_HANDLE} 在 Slack 中: {inputs["draft_reply"]}'
    return {
        "slack_data": ZapierNLARunAction(
            action_id=action["id"],
            zapier_description=action["description"],
            params_schema=action["params"],
        ).run(instructions)
    }


slack_chain = TransformChain(
    input_variables=["draft_reply"],
    output_variables=["slack_data"],
    transform=nla_slack,
)
```

```python
## 最后，执行

overall_chain = SimpleSequentialChain(
    chains=[gmail_chain, reply_chain, slack_chain], verbose=True
)
overall_chain.run(GMAIL_SEARCH_INSTRUCTIONS)
```
```output


[1m> 进入新的 SimpleSequentialChain 链...[0m
[36;1m[1;3m{"from__name": "Silicon Valley Bridge Bank, N.A.", "from__email": "sreply@svb.com", "body_plain": "亲爱的客户，经过混乱、动荡和压力重重的日子后，我们对 SVB 的路径有了清晰的认识，FDIC 完全保障所有存款，并且在我们重建时有一个请求给客户和合作伙伴。Tim Mayopoulos <https://eml.svb.com/NjEwLUtBSy0yNjYAAAGKgoxUeBCLAyF_NxON97X4rKEaNBLG", "reply_to__email": "sreply@svb.com", "subject": "认识新 CEO Tim Mayopoulos", "date": "2023年3月14日 星期二 23:42:29 -0500 (CDT)", "message_url": "https://mail.google.com/mail/u/0/#inbox/186e393b13cfdf0a", "attachment_count": "0", "to__emails": "ankush@langchain.dev", "message_id": "186e393b13cfdf0a", "labels": "重要, 更新类别, 收件箱"}[0m
[33;1m[1;3m
亲爱的硅谷桥银行， 

感谢您的电子邮件以及关于您新任 CEO Tim Mayopoulos 的更新。我们感谢您保持客户和合作伙伴知情的奉献精神，并期待继续与您保持关系。 

此致， 
[您的名字][0m
[38;5;200m[1;3m{"message__text": "亲爱的硅谷桥银行， \\n\\n感谢您的电子邮件以及关于您新任 CEO Tim Mayopoulos 的更新。我们感谢您保持客户和合作伙伴知情的奉献精神，并期待继续与您保持关系。 \\n\\n此致， \\n[您的名字]", "message__permalink": "https://langchain.slack.com/archives/D04TKF5BBHU/p1678859968241629", "channel": "D04TKF5BBHU", "message__bot_profile__name": "Zapier", "message__team": "T04F8K3FZB5", "message__bot_id": "B04TRV4R74K", "message__bot_profile__deleted": "false", "message__bot_profile__app_id": "A024R9PQM", "ts_time": "2023-03-15T05:59:28Z", "message__blocks[]block_id": "p7i", "message__blocks[]elements[]elements[]type": "[['text']]", "message__blocks[]elements[]type": "['rich_text_section']"}[0m

[1m> 完成链。[0m
```
```output
'{"message__text": "亲爱的硅谷桥银行， \\n\\n感谢您的电子邮件以及关于您新任 CEO Tim Mayopoulos 的更新。我们感谢您保持客户和合作伙伴知情的奉献精神，并期待继续与您保持关系。 \\n\\n此致， \\n[您的名字]", "message__permalink": "https://langchain.slack.com/archives/D04TKF5BBHU/p1678859968241629", "channel": "D04TKF5BBHU", "message__bot_profile__name": "Zapier", "message__team": "T04F8K3FZB5", "message__bot_id": "B04TRV4R74K", "message__bot_profile__deleted": "false", "message__bot_profile__app_id": "A024R9PQM", "ts_time": "2023-03-15T05:59:28Z", "message__blocks[]block_id": "p7i", "message__blocks[]elements[]elements[]type": "[[\'text\']]", "message__blocks[]elements[]type": "[\'rich_text_section\']"}'
```

## <a id="oauth">使用 OAuth 访问令牌的示例</a>
以下代码片段展示了如何使用获取的 OAuth 访问令牌初始化包装器。请注意传入的参数与设置环境变量的区别。请查看 [认证文档](https://nla.zapier.com/docs/authentication/#oauth-credentials) 以获取完整的面向用户的 oauth 开发者支持。

开发者的任务是处理 OAuth 握手，以获取和刷新访问令牌。

```python
llm = OpenAI(temperature=0)
zapier = ZapierNLAWrapper(zapier_nla_oauth_access_token="<fill in access token here>")
toolkit = ZapierToolkit.from_zapier_nla_wrapper(zapier)
agent = initialize_agent(
    toolkit.get_tools(), llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)

agent.run(
    "Summarize the last email I received regarding Silicon Valley Bank. Send the summary to the #test-zapier channel in slack."
)
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)