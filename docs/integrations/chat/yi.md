---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/yi.ipynb
---

# ChatYI

这将帮助您开始使用 Yi [聊天模型](/docs/concepts/#chat-models)。有关所有 ChatYi 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/chat_models/lanchain_community.chat_models.yi.ChatYi.html)。

[01.AI](https://www.lingyiwanwu.com/en)，由李开复博士创立，是一家处于 AI 2.0 前沿的全球公司。他们提供尖端的大型语言模型，包括 Yi 系列，参数范围从 6B 到数百亿。01.AI 还提供多模态模型、开放 API 平台以及开源选项，如 Yi-34B/9B/6B 和 Yi-VL。

## 概述

### 集成细节


| 类别 | 包 | 本地 | 可序列化 | JS 支持 | 包下载量 | 包最新版本 |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [ChatYi](https://api.python.langchain.com/en/latest/chat_models/lanchain_community.chat_models.yi.ChatYi.html) | [langchain_community](https://api.python.langchain.com/en/latest/community_api_reference.html) | ✅ | ❌ | ❌ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain_community?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain_community?style=flat-square&label=%20) |

### 模型特性
| [工具调用](/docs/how_to/tool_calling) | [结构化输出](/docs/how_to/structured_output/) | JSON 模式 | [图像输入](/docs/how_to/multimodal_inputs/) | 音频输入 | 视频输入 | [令牌级流式传输](/docs/how_to/chat_streaming/) | 原生异步 | [令牌使用](/docs/how_to/chat_token_usage_tracking/) | [对数概率](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ |

## 设置

要访问 ChatYi 模型，您需要创建一个 01.AI 账户，获取一个 API 密钥，并安装 `langchain_community` 集成包。

### 凭证

前往 [01.AI](https://platform.01.ai) 注册 01.AI 并生成 API 密钥。完成后设置 `YI_API_KEY` 环境变量：

```python
import getpass
import os

os.environ["YI_API_KEY"] = getpass.getpass("Enter your Yi API key: ")
```

如果您想要自动跟踪模型调用，可以通过取消注释以下内容来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：

```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

### 安装

LangChain __ModuleName__ 集成位于 `langchain_community` 包中：

```python
%pip install -qU langchain_community
```

## 实例化

现在我们可以实例化我们的模型对象并生成聊天补全：

- TODO: 使用相关参数更新模型实例化。


```python
from langchain_community.chat_models.yi import ChatYi

llm = ChatYi(
    model="yi-large",
    temperature=0,
    timeout=60,
    yi_api_base="https://api.01.ai/v1/chat/completions",
    # other params...
)
```

## 调用



```python
from langchain_core.messages import HumanMessage, SystemMessage

messages = [
    SystemMessage(content="You are an AI assistant specializing in technology trends."),
    HumanMessage(
        content="What are the potential applications of large language models in healthcare?"
    ),
]

ai_msg = llm.invoke(messages)
ai_msg
```

## 链接

我们可以 [链式](/docs/how_to/sequence/) 使用一个提示模板，如下所示：


```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant that translates {input_language} to {output_language}.",
        ),
        ("human", "{input}"),
    ]
)

chain = prompt | llm
chain.invoke(
    {
        "input_language": "English",
        "output_language": "German",
        "input": "I love programming.",
    }
)
```



```output
AIMessage(content='Ich liebe das Programmieren.', response_metadata={'token_usage': {'completion_tokens': 8, 'prompt_tokens': 33, 'total_tokens': 41}, 'model': 'yi-large'}, id='run-daa3bc58-8289-4d72-a24e-80622fa90d6d-0')
```

## API 参考

有关所有 ChatYi 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/chat_models/langchain_community.chat_models.yi.ChatYi.html

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)