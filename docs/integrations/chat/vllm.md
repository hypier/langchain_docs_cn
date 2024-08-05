---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/vllm.ipynb
sidebar_label: vLLM 聊天
---

# vLLM 聊天

vLLM 可以作为一个服务器部署，模拟 OpenAI API 协议。这允许 vLLM 作为使用 OpenAI API 的应用程序的替代品。该服务器可以使用与 OpenAI API 相同的格式进行查询。

## 概述
这将帮助您开始使用 vLLM [聊天模型](/docs/concepts/#chat-models)，该模型利用了 `langchain-openai` 包。有关所有 `ChatOpenAI` 功能和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.base.ChatOpenAI.html)。

### 集成详情

| 类别 | 包 | 本地 | 可序列化 | JS 支持 | 包下载量 | 包最新版本 |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [ChatOpenAI](https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.base.ChatOpenAI.html) | [langchain_openai](https://api.python.langchain.com/en/latest/langchain_openai.html) | ✅ | beta | ❌ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain_openai?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain_openai?style=flat-square&label=%20) |

### 模型特性
具体的模型特性——例如工具调用、对多模态输入的支持、对令牌级流式传输的支持等——将取决于托管模型。

## 设置

请参阅 vLLM 文档 [这里](https://docs.vllm.ai/en/latest/)。

要通过 LangChain 访问 vLLM 模型，您需要安装 `langchain-openai` 集成包。

### 凭证

身份验证将依赖于推理服务器的具体情况。

如果您想获取模型调用的自动追踪，可以通过取消下面的注释来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：


```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
```

### 安装

LangChain vLLM 集成可以通过 `langchain-openai` 包访问：


```python
%pip install -qU langchain-openai
```

## 实例化

现在我们可以实例化我们的模型对象并生成聊天完成内容：


```python
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_core.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
    SystemMessagePromptTemplate,
)
from langchain_openai import ChatOpenAI
```


```python
inference_server_url = "http://localhost:8000/v1"

llm = ChatOpenAI(
    model="mosaicml/mpt-7b",
    openai_api_key="EMPTY",
    openai_api_base=inference_server_url,
    max_tokens=5,
    temperature=0,
)
```

## 调用


```python
messages = [
    SystemMessage(
        content="You are a helpful assistant that translates English to Italian."
    ),
    HumanMessage(
        content="Translate the following sentence from English to Italian: I love programming."
    ),
]
llm.invoke(messages)
```



```output
AIMessage(content=' Io amo programmare', additional_kwargs={}, example=False)
```

## 链接

我们可以使用提示模板来[链接](/docs/how_to/sequence/)我们的模型，如下所示：

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate(
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

## API 参考

有关通过 `langchain-openai` 提供的所有功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/chat_models/langchain_openai.chat_models.base.ChatOpenAI.html

同时参考 vLLM [文档](https://docs.vllm.ai/en/latest/)。

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)