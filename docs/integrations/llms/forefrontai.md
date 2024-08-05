---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/forefrontai.ipynb
---

# ForefrontAI


`Forefront`平台使您能够微调和使用[开源大型语言模型](https://docs.forefront.ai/forefront/master/models)。

本笔记本介绍如何将Langchain与[ForefrontAI](https://www.forefront.ai/)结合使用。

## 导入


```python
import os

from langchain.chains import LLMChain
from langchain_community.llms import ForefrontAI
from langchain_core.prompts import PromptTemplate
```

## 设置环境 API 密钥
确保从 ForefrontAI 获取您的 API 密钥。您可以获得 5 天的免费试用，以测试不同的模型。

```python
# get a new token: https://docs.forefront.ai/forefront/api-reference/authentication

from getpass import getpass

FOREFRONTAI_API_KEY = getpass()
```

```python
os.environ["FOREFRONTAI_API_KEY"] = FOREFRONTAI_API_KEY
```

## 创建 ForefrontAI 实例
您可以指定不同的参数，例如模型端点 URL、长度、温度等。您必须提供一个端点 URL。

```python
llm = ForefrontAI(endpoint_url="YOUR ENDPOINT URL HERE")
```

## 创建提示模板
我们将为问答创建一个提示模板。

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