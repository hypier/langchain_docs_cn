---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/gooseai.ipynb
---

# GooseAI

`GooseAI` 是一个完全托管的 NLP-as-a-Service，通过 API 提供。GooseAI 提供对 [这些模型](https://goose.ai/docs/models) 的访问。

本笔记本介绍如何将 Langchain 与 [GooseAI](https://goose.ai/) 一起使用。

## 安装 openai
使用 GooseAI API 需要 `openai` 包。使用 `pip install openai` 安装 `openai`。

```python
%pip install --upgrade --quiet  langchain-openai
```

## 导入


```python
import os

from langchain.chains import LLMChain
from langchain_community.llms import GooseAI
from langchain_core.prompts import PromptTemplate
```

## 设置环境 API 密钥
确保从 GooseAI 获取您的 API 密钥。您将获得 $10 的免费积分以测试不同的模型。

```python
from getpass import getpass

GOOSEAI_API_KEY = getpass()
```

```python
os.environ["GOOSEAI_API_KEY"] = GOOSEAI_API_KEY
```

## 创建 GooseAI 实例
您可以指定不同的参数，例如模型名称、生成的最大令牌、温度等。

```python
llm = GooseAI()
```

## 创建提示模板
我们将创建一个用于问答的提示模板。

```python
template = """Question: {question}

Answer: Let's think step by step."""

prompt = PromptTemplate.from_template(template)
```

## 初始化 LLMChain


```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

## 运行 LLMChain
提供一个问题并运行 LLMChain。

```python
question = "What NFL team won the Super Bowl in the year Justin Beiber was born?"

llm_chain.run(question)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)