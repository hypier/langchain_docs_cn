---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/cerebriumai.ipynb
---

# CerebriumAI

`Cerebrium` 是 AWS Sagemaker 的替代方案。它还提供对 [多个 LLM 模型](https://docs.cerebrium.ai/cerebrium/prebuilt-models/deployment) 的 API 访问。

本笔记本介绍如何将 Langchain 与 [CerebriumAI](https://docs.cerebrium.ai/introduction) 一起使用。

## 安装 cerebrium
要使用 `CerebriumAI` API，必须安装 `cerebrium` 包。使用 `pip3 install cerebrium` 安装 `cerebrium`。

```python
# Install the package
!pip3 install cerebrium
```

## 导入


```python
import os

from langchain.chains import LLMChain
from langchain_community.llms import CerebriumAI
from langchain_core.prompts import PromptTemplate
```

## 设置环境 API 密钥
确保从 CerebriumAI 获取您的 API 密钥。请参见 [这里](https://dashboard.cerebrium.ai/login)。您可以获得 1 小时的无服务器 GPU 计算免费时间，以测试不同的模型。


```python
os.environ["CEREBRIUMAI_API_KEY"] = "YOUR_KEY_HERE"
```

## 创建CerebriumAI实例
您可以指定不同的参数，例如模型端点URL、最大长度、温度等。您必须提供一个端点URL。

```python
llm = CerebriumAI(endpoint_url="YOUR ENDPOINT URL HERE")
```

## 创建一个提示模板
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