---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/logprobs.ipynb
---

# 如何获取日志概率

:::info 先决条件

本指南假设您对以下概念有所了解：
- [聊天模型](/docs/concepts/#chat-models)

:::

某些聊天模型可以配置为返回表示给定 token 可能性的 token 级日志概率。本指南将介绍如何在 LangChain 中获取此信息。

## OpenAI

安装 LangChain x OpenAI 包并设置您的 API 密钥


```python
%pip install -qU langchain-openai
```


```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()
```

为了让 OpenAI API 返回日志概率，我们需要配置 `logprobs=True` 参数。然后，logprobs 将作为 `response_metadata` 的一部分包含在每个输出的 [`AIMessage`](https://api.python.langchain.com/en/latest/messages/langchain_core.messages.ai.AIMessage.html) 中：


```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-3.5-turbo-0125").bind(logprobs=True)

msg = llm.invoke(("human", "how are you today"))

msg.response_metadata["logprobs"]["content"][:5]
```



```output
[{'token': 'I', 'bytes': [73], 'logprob': -0.26341408, 'top_logprobs': []},
 {'token': "'m",
  'bytes': [39, 109],
  'logprob': -0.48584133,
  'top_logprobs': []},
 {'token': ' just',
  'bytes': [32, 106, 117, 115, 116],
  'logprob': -0.23484154,
  'top_logprobs': []},
 {'token': ' a',
  'bytes': [32, 97],
  'logprob': -0.0018291725,
  'top_logprobs': []},
 {'token': ' computer',
  'bytes': [32, 99, 111, 109, 112, 117, 116, 101, 114],
  'logprob': -0.052299336,
  'top_logprobs': []}]
```


并且也是流式消息块的一部分：


```python
ct = 0
full = None
for chunk in llm.stream(("human", "how are you today")):
    if ct < 5:
        full = chunk if full is None else full + chunk
        if "logprobs" in full.response_metadata:
            print(full.response_metadata["logprobs"]["content"])
    else:
        break
    ct += 1
```
```output
[]
[{'token': 'I', 'bytes': [73], 'logprob': -0.26593843, 'top_logprobs': []}]
[{'token': 'I', 'bytes': [73], 'logprob': -0.26593843, 'top_logprobs': []}, {'token': "'m", 'bytes': [39, 109], 'logprob': -0.3238896, 'top_logprobs': []}]
[{'token': 'I', 'bytes': [73], 'logprob': -0.26593843, 'top_logprobs': []}, {'token': "'m", 'bytes': [39, 109], 'logprob': -0.3238896, 'top_logprobs': []}, {'token': ' just', 'bytes': [32, 106, 117, 115, 116], 'logprob': -0.23778509, 'top_logprobs': []}]
[{'token': 'I', 'bytes': [73], 'logprob': -0.26593843, 'top_logprobs': []}, {'token': "'m", 'bytes': [39, 109], 'logprob': -0.3238896, 'top_logprobs': []}, {'token': ' just', 'bytes': [32, 106, 117, 115, 116], 'logprob': -0.23778509, 'top_logprobs': []}, {'token': ' a', 'bytes': [32, 97], 'logprob': -0.0022134194, 'top_logprobs': []}]
```

## 下一步

您现在已经学习了如何在 LangChain 中从 OpenAI 模型获取 logprobs。

接下来，请查看本节中其他关于聊天模型的操作指南，例如 [如何获取模型返回结构化输出](/docs/how_to/structured_output) 或 [如何跟踪令牌使用情况](/docs/how_to/chat_token_usage_tracking)。