---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/snowflake.ipynb
---

# Snowflake Cortex

[Snowflake Cortex](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions) 让您即时访问行业领先的大型语言模型（LLMs），这些模型由 Mistral、Reka、Meta 和 Google 等公司的研究人员训练，包括 [Snowflake Arctic](https://www.snowflake.com/en/data-cloud/arctic/)，这是一个由 Snowflake 开发的开源企业级模型。

本示例介绍如何使用 LangChain 与 Snowflake Cortex 进行交互。

### 安装与设置

我们首先通过下面的命令安装 `snowflake-snowpark-python` 库。然后，我们将凭证配置为环境变量，或直接传递它们。

```python
%pip install --upgrade --quiet snowflake-snowpark-python
```
```output
注意：您可能需要重启内核以使用更新的包。
```

```python
import getpass
import os

# 第一步是设置环境变量，以连接到 Snowflake，
# 您也可以在实例化模型时传递这些 Snowflake 凭证

if os.environ.get("SNOWFLAKE_ACCOUNT") is None:
    os.environ["SNOWFLAKE_ACCOUNT"] = getpass.getpass("账户: ")

if os.environ.get("SNOWFLAKE_USERNAME") is None:
    os.environ["SNOWFLAKE_USERNAME"] = getpass.getpass("用户名: ")

if os.environ.get("SNOWFLAKE_PASSWORD") is None:
    os.environ["SNOWFLAKE_PASSWORD"] = getpass.getpass("密码: ")

if os.environ.get("SNOWFLAKE_DATABASE") is None:
    os.environ["SNOWFLAKE_DATABASE"] = getpass.getpass("数据库: ")

if os.environ.get("SNOWFLAKE_SCHEMA") is None:
    os.environ["SNOWFLAKE_SCHEMA"] = getpass.getpass("模式: ")

if os.environ.get("SNOWFLAKE_WAREHOUSE") is None:
    os.environ["SNOWFLAKE_WAREHOUSE"] = getpass.getpass("仓库: ")

if os.environ.get("SNOWFLAKE_ROLE") is None:
    os.environ["SNOWFLAKE_ROLE"] = getpass.getpass("角色: ")
```


```python
from langchain_community.chat_models import ChatSnowflakeCortex
from langchain_core.messages import HumanMessage, SystemMessage

# 默认情况下，我们将使用提供的模型：`snowflake-arctic`，功能：`complete`
chat = ChatSnowflakeCortex()
```

上面的单元假定您的 Snowflake 凭证已设置在环境变量中。如果您更愿意手动指定它们，请使用以下代码：

```python
chat = ChatSnowflakeCortex(
    # 更改默认的 cortex 模型和功能
    model="snowflake-arctic",
    cortex_function="complete",

    # 更改默认生成参数
    temperature=0,
    max_tokens=10,
    top_p=0.95,

    # 指定 Snowflake 凭证
    account="YOUR_SNOWFLAKE_ACCOUNT",
    username="YOUR_SNOWFLAKE_USERNAME",
    password="YOUR_SNOWFLAKE_PASSWORD",
    database="YOUR_SNOWFLAKE_DATABASE",
    schema="YOUR_SNOWFLAKE_SCHEMA",
    role="YOUR_SNOWFLAKE_ROLE",
    warehouse="YOUR_SNOWFLAKE_WAREHOUSE"
)
```

### 调用模型
我们现在可以使用 `invoke` 或 `generate` 方法来调用模型。

#### 生成


```python
messages = [
    SystemMessage(content="You are a friendly assistant."),
    HumanMessage(content="What are large language models?"),
]
chat.invoke(messages)
```



```output
AIMessage(content=" Large language models are artificial intelligence systems designed to understand, generate, and manipulate human language. These models are typically based on deep learning techniques and are trained on vast amounts of text data to learn patterns and structures in language. They can perform a wide range of language-related tasks, such as language translation, text generation, sentiment analysis, and answering questions. Some well-known large language models include Google's BERT, OpenAI's GPT series, and Facebook's RoBERTa. These models have shown remarkable performance in various natural language processing tasks, and their applications continue to expand as research in AI progresses.", response_metadata={'completion_tokens': 131, 'prompt_tokens': 29, 'total_tokens': 160}, id='run-5435bd0a-83fd-4295-b237-66cbd1b5c0f3-0')
```

### 流媒体
`ChatSnowflakeCortex` 目前不支持流媒体。对流媒体的支持将在后续版本中推出！

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)