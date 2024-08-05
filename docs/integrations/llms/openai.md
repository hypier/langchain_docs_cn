---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/openai.ipynb
---

# OpenAI

:::caution
您当前正在查看有关使用 OpenAI [文本补全模型](/docs/concepts/#llms) 的文档。最新和最受欢迎的 OpenAI 模型是 [聊天补全模型](/docs/concepts/#chat-models)。

除非您正在使用 `gpt-3.5-turbo-instruct`，否则您可能想要查看 [此页面](/docs/integrations/chat/openai/)。
:::

[OpenAI](https://platform.openai.com/docs/introduction) 提供了一系列适用于不同任务的不同能力模型。

此示例介绍了如何使用 LangChain 与 `OpenAI` [模型](https://platform.openai.com/docs/models) 进行交互。


```python
# get a token: https://platform.openai.com/account/api-keys

from getpass import getpass

OPENAI_API_KEY = getpass()
```


```python
import os

os.environ["OPENAI_API_KEY"] = OPENAI_API_KEY
```

如果您需要指定您的组织 ID，可以使用以下单元格。然而，如果您仅属于一个组织或打算使用默认组织，则不需要此操作。您可以在 [这里](https://platform.openai.com/account/api-keys) 检查您的默认组织。

要指定您的组织，您可以使用以下代码：
```python
OPENAI_ORGANIZATION = getpass()

os.environ["OPENAI_ORGANIZATION"] = OPENAI_ORGANIZATION
```


```python
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI
```


```python
template = """Question: {question}

Answer: Let's think step by step."""

prompt = PromptTemplate.from_template(template)
```


```python
llm = OpenAI()
```

如果您想手动指定您的 OpenAI API 密钥和/或组织 ID，可以使用以下代码：
```python
llm = OpenAI(openai_api_key="YOUR_API_KEY", openai_organization="YOUR_ORGANIZATION_ID")
```
如果不适用，请删除 openai_organization 参数。


```python
llm_chain = prompt | llm
```


```python
question = "What NFL team won the Super Bowl in the year Justin Beiber was born?"

llm_chain.invoke(question)
```



```output
' Justin Bieber was born on March 1, 1994. The Super Bowl is typically played in late January or early February. So, we need to look at the Super Bowl from 1994. In 1994, the Super Bowl was Super Bowl XXVIII, played on January 30, 1994. The winning team of that Super Bowl was the Dallas Cowboys.'
```


如果您处于显式代理后面，可以指定 http_client 进行传递。


```python
pip install httpx

import httpx

openai = OpenAI(model_name="gpt-3.5-turbo-instruct", http_client=httpx.Client(proxies="http://proxy.yourcompany.com:8080"))
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)