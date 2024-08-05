---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/moonshot.ipynb
---

# MoonshotChat

[Moonshot](https://platform.moonshot.cn/) 是一家提供 LLM 服务的中国初创公司，面向企业和个人。

本示例介绍如何使用 LangChain 与 Moonshot 进行交互。


```python
from langchain_community.llms.moonshot import Moonshot
```


```python
import os

# 从以下网址生成您的 API 密钥: https://platform.moonshot.cn/console/api-keys
os.environ["MOONSHOT_API_KEY"] = "MOONSHOT_API_KEY"
```


```python
llm = Moonshot()
# 或使用特定模型
# 可用模型: https://platform.moonshot.cn/docs
# llm = Moonshot(model="moonshot-v1-128k")
```


```python
# 提示模型
llm.invoke("What is the difference between panda and bear?")
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)