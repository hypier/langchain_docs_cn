---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat_loaders/gmail.ipynb
---

# GMail

此加载器介绍了如何从 GMail 加载数据。您可能希望通过多种方式从 GMail 加载数据。该加载器目前在执行此操作时有明确的偏好。它的工作方式是首先查找您发送的所有邮件。然后查找您回复的邮件。接着，它获取那封之前的邮件，并创建一个训练示例，该示例包括之前的邮件和您的回复邮件。

请注意，这里存在明显的限制。例如，所有创建的示例仅查看之前的邮件以获取上下文。

使用方法：

- 设置 Google 开发者账户：访问 Google 开发者控制台，创建一个项目，并为该项目启用 Gmail API。这将为您提供一个稍后需要的 credentials.json 文件。

- 安装 Google 客户端库：运行以下命令以安装 Google 客户端库：

```python
%pip install --upgrade --quiet  google-auth google-auth-oauthlib google-auth-httplib2 google-api-python-client
```

```python
import os.path

from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow

SCOPES = ["https://www.googleapis.com/auth/gmail.readonly"]

creds = None
# 文件 token.json 存储用户的访问和刷新令牌，并且在授权流程第一次完成时自动创建。
if os.path.exists("email_token.json"):
    creds = Credentials.from_authorized_user_file("email_token.json", SCOPES)
# 如果没有可用的（有效的）凭据，则让用户登录。
if not creds or not creds.valid:
    if creds and creds.expired and creds.refresh_token:
        creds.refresh(Request())
    else:
        flow = InstalledAppFlow.from_client_secrets_file(
            # 在此处放置您的凭据文件。请按照此处的说明创建 json 文件 https://cloud.google.com/docs/authentication/getting-started
            "creds.json",
            SCOPES,
        )
        creds = flow.run_local_server(port=0)
    # 将凭据保存以供下次使用
    with open("email_token.json", "w") as token:
        token.write(creds.to_json())
```

```python
from langchain_community.chat_loaders.gmail import GMailLoader
```

```python
loader = GMailLoader(creds=creds, n=3)
```

```python
data = loader.load()
```

```python
# 有时可能会出现错误，我们会默默忽略
len(data)
```

```output
2
```

```python
from langchain_community.chat_loaders.utils import (
    map_ai_messages,
)
```

```python
# 这使得由 hchase@langchain.com 发送的消息成为 AI 消息
# 这意味着您将训练一个 LLM 来预测其作为 hchase 的回复
training_data = list(
    map_ai_messages(data, sender="Harrison Chase <hchase@langchain.com>")
)
```