---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/twilio.ipynb
---

# Twilio

本笔记本介绍如何使用 [Twilio](https://www.twilio.com) API 封装通过 SMS 或 [Twilio Messaging Channels](https://www.twilio.com/docs/messaging/channels) 发送消息。

Twilio Messaging Channels 便于与第三方消息应用集成，并允许您通过 WhatsApp Business Platform (GA)、Facebook Messenger (Public Beta) 和 Google Business Messages (Private Beta) 发送消息。

## 设置

要使用此工具，您需要安装 Python Twilio 包 `twilio` 

```python
%pip install --upgrade --quiet  twilio
```

您还需要设置一个 Twilio 帐户并获取您的凭据。您需要您的帐户字符串标识符（SID）和您的身份验证令牌（Auth Token）。您还需要一个发送消息的号码。

您可以将这些作为命名参数 `account_sid`、`auth_token`、`from_number` 传递给 TwilioAPIWrapper，或者您可以设置环境变量 `TWILIO_ACCOUNT_SID`、`TWILIO_AUTH_TOKEN`、`TWILIO_FROM_NUMBER`。

## 发送短信


```python
from langchain_community.utilities.twilio import TwilioAPIWrapper
```


```python
twilio = TwilioAPIWrapper(
    #     account_sid="foo",
    #     auth_token="bar",
    #     from_number="baz,"
)
```


```python
twilio.run("hello world", "+16162904619")
```

## 发送 WhatsApp 消息

您需要将您的 WhatsApp 商业账户与 Twilio 关联。您还需要确保发送消息的号码已在 Twilio 上配置为 WhatsApp 启用发送者，并已在 WhatsApp 上注册。

```python
from langchain_community.utilities.twilio import TwilioAPIWrapper
```

```python
twilio = TwilioAPIWrapper(
    #     account_sid="foo",
    #     auth_token="bar",
    #     from_number="whatsapp: baz,"
)
```

```python
twilio.run("hello world", "whatsapp: +16162904619")
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)