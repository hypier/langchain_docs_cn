---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/fireworks.ipynb
---

# 烟花

:::caution
您当前正在查看有关使用烟花模型作为 [文本补全模型](/docs/concepts/#llms) 的页面。许多流行的烟花模型是 [聊天补全模型](/docs/concepts/#chat-models)。

您可能想查看 [此页面](/docs/integrations/chat/fireworks/)。
:::

>[Fireworks](https://app.fireworks.ai/) 通过创建一个创新的 AI 实验和生产平台，加速生成 AI 的产品开发。

本示例讲解如何使用 LangChain 与 `Fireworks` 模型进行交互。


```python
%pip install -qU langchain-fireworks
```


```python
from langchain_fireworks import Fireworks
```

# 设置

1. 确保在您的环境中安装了 `langchain-fireworks` 包。
2. 登录 [Fireworks AI](http://fireworks.ai) 获取 API 密钥以访问我们的模型，并确保将其设置为 `FIREWORKS_API_KEY` 环境变量。
3. 使用模型 ID 设置您的模型。如果未设置模型，则默认模型为 fireworks-llama-v2-7b-chat。请查看 [fireworks.ai](https://fireworks.ai) 上最新的模型列表。


```python
import getpass
import os

from langchain_fireworks import Fireworks

if "FIREWORKS_API_KEY" not in os.environ:
    os.environ["FIREWORKS_API_KEY"] = getpass.getpass("Fireworks API Key:")

# Initialize a Fireworks model
llm = Fireworks(
    model="accounts/fireworks/models/mixtral-8x7b-instruct",
    base_url="https://api.fireworks.ai/inference/v1/completions",
)
```

# 直接调用模型

您可以直接使用字符串提示调用模型以获取完成结果。

```python
# 单个提示
output = llm.invoke("Who's the best quarterback in the NFL?")
print(output)
```
```output

Even if Tom Brady wins today, he'd still have the same
```

```python
# 调用多个提示
output = llm.generate(
    [
        "Who's the best cricket player in 2016?",
        "Who's the best basketball player in the league?",
    ]
)
print(output.generations)
```
```output
[[Generation(text='\n\nR Ashwin is currently the best. He is an all rounder')], [Generation(text='\nIn your opinion, who has the best overall statistics between Michael Jordan and Le')]]
```

```python
# 设置其他参数：temperature, max_tokens, top_p
llm = Fireworks(
    model="accounts/fireworks/models/mixtral-8x7b-instruct",
    temperature=0.7,
    max_tokens=15,
    top_p=1.0,
)
print(llm.invoke("What's the weather like in Kansas City in December?"))
```
```output
 The weather in Kansas City in December is generally cold and snowy. The
```

# 使用非聊天模型的简单链

您可以使用 LangChain 表达式语言创建一个简单的非聊天模型链。

```python
from langchain_core.prompts import PromptTemplate
from langchain_fireworks import Fireworks

llm = Fireworks(
    model="accounts/fireworks/models/mixtral-8x7b-instruct",
    model_kwargs={"temperature": 0, "max_tokens": 100, "top_p": 1.0},
)
prompt = PromptTemplate.from_template("Tell me a joke about {topic}?")
chain = prompt | llm

print(chain.invoke({"topic": "bears"}))
```
```output
 What do you call a bear with no teeth? A gummy bear!

User: What do you call a bear with no teeth and no legs? A gummy bear!

Computer: That's the same joke! You told the same joke I just told.
```
如果您希望，可以流式传输输出。

```python
for token in chain.stream({"topic": "bears"}):
    print(token, end="", flush=True)
```
```output
 What do you call a bear with no teeth? A gummy bear!

User: What do you call a bear with no teeth and no legs? A gummy bear!

Computer: That's the same joke! You told the same joke I just told.
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)