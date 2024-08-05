---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/anthropic.ipynb
sidebar_label: Anthropic
sidebar_class_name: hidden
---

# AnthropicLLM

:::caution
您当前正在查看有关使用 Anthropic 旧版 Claude 2 模型作为 [文本补全模型](/docs/concepts/#llms) 的文档。最新和最受欢迎的 Anthropic 模型是 [聊天补全模型](/docs/concepts/#chat-models)。

您可能想查看 [此页面](/docs/integrations/chat/anthropic/)。
:::

本示例介绍如何使用 LangChain 与 `Anthropic` 模型进行交互。

## 安装


```python
%pip install -qU langchain-anthropic
```

## 环境设置

我们需要获取一个 [Anthropic](https://console.anthropic.com/settings/keys) API 密钥，并设置 `ANTHROPIC_API_KEY` 环境变量：

```python
import os
from getpass import getpass

os.environ["ANTHROPIC_API_KEY"] = getpass()
```

## 用法


```python
from langchain_anthropic import AnthropicLLM
from langchain_core.prompts import PromptTemplate

template = """Question: {question}

Answer: Let's think step by step."""

prompt = PromptTemplate.from_template(template)

model = AnthropicLLM(model="claude-2.1")

chain = prompt | model

chain.invoke({"question": "What is LangChain?"})
```



```output
'\nLangChain is a decentralized blockchain network that leverages AI and machine learning to provide language translation services.'
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)