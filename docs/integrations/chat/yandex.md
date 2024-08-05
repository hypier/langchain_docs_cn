---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/yandex.ipynb
sidebar_label: YandexGPT
---

# ChatYandexGPT

本笔记本介绍了如何使用 Langchain 与 [YandexGPT](https://cloud.yandex.com/en/services/yandexgpt) 聊天模型。

要使用此功能，您需要安装 `yandexcloud` python 包。

```python
%pip install --upgrade --quiet  yandexcloud
```

首先，您应该 [创建服务帐户](https://cloud.yandex.com/en/docs/iam/operations/sa/create)，并赋予 `ai.languageModels.user` 角色。

接下来，您有两种身份验证选项：
- [IAM 令牌](https://cloud.yandex.com/en/docs/iam/operations/iam-token/create-for-sa)。
    您可以在构造函数参数 `iam_token` 中指定令牌，或在环境变量 `YC_IAM_TOKEN` 中指定。

- [API 密钥](https://cloud.yandex.com/en/docs/iam/operations/api-key/create)
    您可以在构造函数参数 `api_key` 中指定密钥，或在环境变量 `YC_API_KEY` 中指定。

要指定模型，您可以使用 `model_uri` 参数，详细信息请参见 [文档](https://cloud.yandex.com/en/docs/yandexgpt/concepts/models#yandexgpt-generation)。

默认情况下，从参数 `folder_id` 或环境变量 `YC_FOLDER_ID` 指定的文件夹中使用最新版本的 `yandexgpt-lite`。

```python
from langchain_community.chat_models import ChatYandexGPT
from langchain_core.messages import HumanMessage, SystemMessage
```

```python
chat_model = ChatYandexGPT()
```

```python
answer = chat_model.invoke(
    [
        SystemMessage(
            content="You are a helpful assistant that translates English to French."
        ),
        HumanMessage(content="I love programming."),
    ]
)
answer
```

```output
AIMessage(content='Je adore le programmement.')
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)