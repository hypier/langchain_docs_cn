---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/minimax.ipynb
---

# Minimax

[Minimax](https://api.minimax.chat) 是一家提供自然语言处理模型的中国初创公司，服务于企业和个人。

本示例演示了如何使用 Langchain 与 Minimax 进行交互。

# 设置

要运行此笔记本，您需要一个 [Minimax 账户](https://api.minimax.chat)、一个 [API 密钥](https://api.minimax.chat/user-center/basic-information/interface-key) 和一个 [组 ID](https://api.minimax.chat/user-center/basic-information)

# 单模型调用


```python
from langchain_community.llms import Minimax
```


```python
# 加载模型
minimax = Minimax(minimax_api_key="YOUR_API_KEY", minimax_group_id="YOUR_GROUP_ID")
```


```python
# 提示模型
minimax("What is the difference between panda and bear?")
```

# 链式模型调用


```python
# get api_key and group_id: https://api.minimax.chat/user-center/basic-information
# We need `MINIMAX_API_KEY` and `MINIMAX_GROUP_ID`

import os

os.environ["MINIMAX_API_KEY"] = "YOUR_API_KEY"
os.environ["MINIMAX_GROUP_ID"] = "YOUR_GROUP_ID"
```


```python
from langchain.chains import LLMChain
from langchain_community.llms import Minimax
from langchain_core.prompts import PromptTemplate
```


```python
template = """Question: {question}

Answer: Let's think step by step."""

prompt = PromptTemplate.from_template(template)
```


```python
llm = Minimax()
```


```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```


```python
question = "What NBA team won the Championship in the year Jay Zhou was born?"

llm_chain.run(question)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)