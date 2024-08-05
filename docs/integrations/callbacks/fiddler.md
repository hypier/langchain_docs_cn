---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/callbacks/fiddler.ipynb
---

# Fiddler

>[Fiddler](https://www.fiddler.ai/) 是企业生成和预测系统运维的先驱，提供一个统一的平台，使数据科学、MLOps、风险、合规、分析和其他业务部门团队能够在企业规模上监控、解释、分析和改进机器学习部署。

## 1. 安装与设置


```python
#!pip install langchain langchain-community langchain-openai fiddler-client
```

## 2. Fiddler 连接详情 

*在您可以使用 Fiddler 添加有关您的模型的信息之前*

1. 您用于连接 Fiddler 的 URL
2. 您的组织 ID
3. 您的授权令牌

这些信息可以通过导航到您的 Fiddler 环境的 *设置* 页面找到。


```python
URL = ""  # Your Fiddler instance URL, Make sure to include the full URL (including https://). For example: https://demo.fiddler.ai
ORG_NAME = ""
AUTH_TOKEN = ""  # Your Fiddler instance auth token

# Fiddler project and model names, used for model registration
PROJECT_NAME = ""
MODEL_NAME = ""  # Model name in Fiddler
```

## 3. 创建 Fiddler 回调处理程序实例


```python
from langchain_community.callbacks.fiddler_callback import FiddlerCallbackHandler

fiddler_handler = FiddlerCallbackHandler(
    url=URL,
    org=ORG_NAME,
    project=PROJECT_NAME,
    model=MODEL_NAME,
    api_key=AUTH_TOKEN,
)
```

## 示例 1 : 基本链

```python
from langchain_core.output_parsers import StrOutputParser
from langchain_openai import OpenAI

# 注意 : 确保在环境变量 OPENAI_API_KEY 中设置 openai API 密钥
llm = OpenAI(temperature=0, streaming=True, callbacks=[fiddler_handler])
output_parser = StrOutputParser()

chain = llm | output_parser

# 调用链。调用将记录到 Fiddler，并自动生成指标
chain.invoke("How far is moon from earth?")
```

```python
# 另外几个调用
chain.invoke("What is the temperature on Mars?")
chain.invoke("How much is 2 + 200000?")
chain.invoke("Which movie won the oscars this year?")
chain.invoke("Can you write me a poem about insomnia?")
chain.invoke("How are you doing today?")
chain.invoke("What is the meaning of life?")
```

## 示例 2 : 带有提示模板的链

```python
from langchain_core.prompts import (
    ChatPromptTemplate,
    FewShotChatMessagePromptTemplate,
)

examples = [
    {"input": "2+2", "output": "4"},
    {"input": "2+3", "output": "5"},
]

example_prompt = ChatPromptTemplate.from_messages(
    [
        ("human", "{input}"),
        ("ai", "{output}"),
    ]
)

few_shot_prompt = FewShotChatMessagePromptTemplate(
    example_prompt=example_prompt,
    examples=examples,
)

final_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "你是一位神奇的数学巫师。"),
        few_shot_prompt,
        ("human", "{input}"),
    ]
)

# 注意 : 确保在环境变量 OPENAI_API_KEY 中设置了 openai API 密钥
llm = OpenAI(temperature=0, streaming=True, callbacks=[fiddler_handler])

chain = final_prompt | llm

# 调用链。调用将被记录到 Fiddler，并自动生成指标
chain.invoke({"input": "三角形的平方是多少？"})
```