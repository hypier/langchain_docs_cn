---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/callbacks/trubrics.ipynb
---

# Trubrics

>[Trubrics](https://trubrics.com) 是一个 LLM 用户分析平台，允许您收集、分析和管理用户对 AI 模型的提示和反馈。
>
>查看 [Trubrics repo](https://github.com/trubrics/trubrics-sdk) 获取有关 `Trubrics` 的更多信息。

在本指南中，我们将介绍如何设置 `TrubricsCallbackHandler`。

## 安装与设置


```python
%pip install --upgrade --quiet  trubrics langchain langchain-community
```

### 获取 Trubrics 凭证

如果您没有 Trubrics 账户，请在 [这里](https://trubrics.streamlit.app/) 创建一个。在本教程中，我们将使用在账户创建时生成的 `default` 项目。

现在将您的凭证设置为环境变量：


```python
import os

os.environ["TRUBRICS_EMAIL"] = "***@***"
os.environ["TRUBRICS_PASSWORD"] = "***"
```


```python
from langchain_community.callbacks.trubrics_callback import TrubricsCallbackHandler
```

### 用法

`TrubricsCallbackHandler` 可以接收各种可选参数。有关可以传递给 Trubrics 提示的 kwargs，请参见 [这里](https://trubrics.github.io/trubrics-sdk/platform/user_prompts/#saving-prompts-to-trubrics)。

```python
class TrubricsCallbackHandler(BaseCallbackHandler):

    """
    Callback handler for Trubrics.
    
    Args:
        project: a trubrics project, default project is "default"
        email: a trubrics account email, can equally be set in env variables
        password: a trubrics account password, can equally be set in env variables
        **kwargs: all other kwargs are parsed and set to trubrics prompt variables, or added to the `metadata` dict
    """
```

## 示例

以下是如何使用 `TrubricsCallbackHandler` 与 Langchain [LLMs](/docs/how_to#llms) 或 [聊天模型](/docs/how_to#chat-models) 的两个示例。我们将使用 OpenAI 模型，因此在此处设置您的 `OPENAI_API_KEY` 密钥：


```python
os.environ["OPENAI_API_KEY"] = "sk-***"
```

### 1. 使用 LLM


```python
from langchain_openai import OpenAI
```


```python
llm = OpenAI(callbacks=[TrubricsCallbackHandler()])
```
```output
[32m2023-09-26 11:30:02.149[0m | [1mINFO    [0m | [36mtrubrics.platform.auth[0m:[36mget_trubrics_auth_token[0m:[36m61[0m - [1m用户 jeff.kayne@trubrics.com 已通过身份验证。[0m
```

```python
res = llm.generate(["Tell me a joke", "Write me a poem"])
```
```output
[32m2023-09-26 11:30:07.760[0m | [1mINFO    [0m | [36mtrubrics.platform[0m:[36mlog_prompt[0m:[36m102[0m - [1m用户提示已保存到 Trubrics。[0m
[32m2023-09-26 11:30:08.042[0m | [1mINFO    [0m | [36mtrubrics.platform[0m:[36mlog_prompt[0m:[36m102[0m - [1m用户提示已保存到 Trubrics。[0m
```

```python
print("--> GPT's joke: ", res.generations[0][0].text)
print()
print("--> GPT's poem: ", res.generations[1][0].text)
```
```output
--> GPT's joke:  

Q: What did the fish say when it hit the wall?
A: Dam!

--> GPT's poem:  

A Poem of Reflection

I stand here in the night,
The stars above me filling my sight.
I feel such a deep connection,
To the world and all its perfection.

A moment of clarity,
The calmness in the air so serene.
My mind is filled with peace,
And I am released.

The past and the present,
My thoughts create a pleasant sentiment.
My heart is full of joy,
My soul soars like a toy.

I reflect on my life,
And the choices I have made.
My struggles and my strife,
The lessons I have paid.

The future is a mystery,
But I am ready to take the leap.
I am ready to take the lead,
And to create my own destiny.
```

### 2. 使用聊天模型


```python
from langchain_core.messages import HumanMessage, SystemMessage
from langchain_openai import ChatOpenAI
```


```python
chat_llm = ChatOpenAI(
    callbacks=[
        TrubricsCallbackHandler(
            project="default",
            tags=["chat model"],
            user_id="user-id-1234",
            some_metadata={"hello": [1, 2]},
        )
    ]
)
```


```python
chat_res = chat_llm.invoke(
    [
        SystemMessage(content="你的每个回答都必须与OpenAI有关。"),
        HumanMessage(content="给我讲个笑话"),
    ]
)
```
```output
[32m2023-09-26 11:30:10.550[0m | [1mINFO    [0m | [36mtrubrics.platform[0m:[36mlog_prompt[0m:[36m102[0m - [1m用户提示已保存到Trubrics。[0m
```

```python
print(chat_res.content)
```
```output
为什么OpenAI的计算机去参加派对？

因为它想见见它的AI朋友们，享受一下乐趣！
```