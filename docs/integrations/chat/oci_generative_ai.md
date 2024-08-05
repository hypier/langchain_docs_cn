---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/oci_generative_ai.ipynb
sidebar_label: OCIGenAI
---

# ChatOCIGenAI

本笔记本提供了有关如何开始使用 OCIGenAI [聊天模型](/docs/concepts/#chat-models) 的快速概述。有关所有 ChatOCIGenAI 特性和配置的详细文档，请访问 [API 参考](https://api.python.langchain.com/en/latest/chat_models/langchain_community.chat_models.oci_generative_ai.ChatOCIGenAI.html)。

Oracle Cloud Infrastructure (OCI) 生成式 AI 是一项完全托管的服务，提供一套最先进的、可定制的大型语言模型 (LLMs)，涵盖广泛的用例，并可以通过单一 API 访问。
使用 OCI 生成式 AI 服务，您可以访问现成的预训练模型，或根据自己的数据在专用 AI 集群上创建和托管自己的微调自定义模型。有关该服务和 API 的详细文档可在 __[这里](https://docs.oracle.com/en-us/iaas/Content/generative-ai/home.htm)__ 和 __[这里](https://docs.oracle.com/en-us/iaas/api/#/en/generative-ai/20231130/)__ 获得。

```python
# This is a code block that remains unchanged
print("Hello, World!")
```

## Overview

### 集成详情

| 类别 | 包 | 本地 | 可序列化 | [JS 支持](https://js.langchain.com/v0.2/docs/integrations/chat/oci_generative_ai) | 包下载量 | 包最新版本 |
| :--- | :--- | :---: | :---: |  :---: | :---: | :---: |
| [ChatOCIGenAI](https://api.python.langchain.com/en/latest/chat_models/langchain_community.chat_models.oci_generative_ai.ChatOCIGenAI.html) | [langchain-community](https://api.python.langchain.com/en/latest/community_api_reference.html) | ❌ | ❌ | ❌ | ![PyPI - Downloads](https://img.shields.io/pypi/dm/langchain-oci-generative-ai?style=flat-square&label=%20) | ![PyPI - Version](https://img.shields.io/pypi/v/langchain-oci-generative-ai?style=flat-square&label=%20) |

### 模型特性
| [工具调用](/docs/how_to/tool_calling/) | [结构化输出](/docs/how_to/structured_output/) | JSON 模式 | [图像输入](/docs/how_to/multimodal_inputs/) | 音频输入 | 视频输入 | [令牌级流式传输](/docs/how_to/chat_streaming/) | 原生异步 | [令牌使用](/docs/how_to/chat_token_usage_tracking/) | [对数概率](/docs/how_to/logprobs/) |
| :---: | :---: | :---: | :---: |  :---: | :---: | :---: | :---: | :---: | :---: |
| ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |

## 设置

要访问 OCIGenAI 模型，您需要安装 `oci` 和 `langchain-community` 包。

### Credentials

此集成支持的凭证和身份验证方法与其他 OCI 服务使用的相同，并遵循 __[标准 SDK 身份验证](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdk_authentication_methods.htm)__ 方法，具体包括 API 密钥、会话令牌、实例主体和资源主体。

API 密钥是上述示例中使用的默认身份验证方法。以下示例演示如何使用不同的身份验证方法（会话令牌）

### 安装

LangChain OCIGenAI 集成位于 `langchain-community` 包中，您还需要安装 `oci` 包：

```python
%pip install -qU langchain-community oci
```

## 实例化

现在我们可以实例化我们的模型对象并生成聊天完成内容：



```python
from langchain_community.chat_models.oci_generative_ai import ChatOCIGenAI
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage

chat = ChatOCIGenAI(
    model_id="cohere.command-r-16k",
    service_endpoint="https://inference.generativeai.us-chicago-1.oci.oraclecloud.com",
    compartment_id="MY_OCID",
    model_kwargs={"temperature": 0.7, "max_tokens": 500},
)
```

## 调用


```python
messages = [
    SystemMessage(content="your are an AI assistant."),
    AIMessage(content="Hi there human!"),
    HumanMessage(content="tell me a joke."),
]
response = chat.invoke(messages)
```


```python
print(response.content)
```

## 链接

我们可以通过一个提示模板来[链接](/docs/how_to/sequence/)我们的模型，如下所示：


```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_template("Tell me a joke about {topic}")
chain = prompt | chat

response = chain.invoke({"topic": "dogs"})
print(response.content)
```

## API 参考

有关所有 ChatOCIGenAI 功能和配置的详细文档，请访问 API 参考： https://api.python.langchain.com/en/latest/chat_models/langchain_community.chat_models.oci_generative_ai.ChatOCIGenAI.html

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)