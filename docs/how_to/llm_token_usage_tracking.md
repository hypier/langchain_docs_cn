---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/llm_token_usage_tracking.ipynb
---

# 如何跟踪 LLM 的令牌使用情况

跟踪令牌使用情况以计算成本是将应用程序投入生产的重要部分。本指南介绍了如何从您的 LangChain 模型调用中获取此信息。

:::info 先决条件

本指南假设您对以下概念有所了解：

- [LLMs](/docs/concepts/#llms)
:::

## 使用 LangSmith

您可以使用 [LangSmith](https://www.langchain.com/langsmith) 来帮助跟踪您 LLM 应用中的令牌使用情况。请参阅 [LangSmith 快速入门指南](https://docs.smith.langchain.com/)。

## 使用回调

有一些特定于 API 的回调上下文管理器，可以让您跟踪多个调用中的令牌使用情况。您需要检查是否有此类集成可用于您的特定模型。

如果您的模型没有此类集成，您可以通过调整 [OpenAI 回调管理器](https://api.python.langchain.com/en/latest/_modules/langchain_community/callbacks/openai_info.html#OpenAICallbackHandler) 的实现来创建自定义回调管理器。

### OpenAI

让我们首先看一个非常简单的示例，来跟踪单个 Chat 模型调用的令牌使用情况。

:::danger

回调处理程序目前不支持对传统语言模型（例如，`langchain_openai.OpenAI`）进行流式令牌计数。有关流式上下文的支持，请参考聊天模型的相应指南 [这里](/docs/how_to/chat_token_usage_tracking)。

:::

### 单次调用


```python
from langchain_community.callbacks import get_openai_callback
from langchain_openai import OpenAI

llm = OpenAI(model_name="gpt-3.5-turbo-instruct")

with get_openai_callback() as cb:
    result = llm.invoke("Tell me a joke")
    print(result)
    print("---")
print()

print(f"Total Tokens: {cb.total_tokens}")
print(f"Prompt Tokens: {cb.prompt_tokens}")
print(f"Completion Tokens: {cb.completion_tokens}")
print(f"Total Cost (USD): ${cb.total_cost}")
```
```output


Why don't scientists trust atoms?

Because they make up everything.
---

Total Tokens: 18
Prompt Tokens: 4
Completion Tokens: 14
Total Cost (USD): $3.4e-05
```

### 多次调用

上下文管理器中的任何内容都会被跟踪。以下是使用它跟踪对链的多个顺序调用的示例。这也适用于可能使用多个步骤的代理。


```python
from langchain_community.callbacks import get_openai_callback
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI

llm = OpenAI(model_name="gpt-3.5-turbo-instruct")

template = PromptTemplate.from_template("Tell me a joke about {topic}")
chain = template | llm

with get_openai_callback() as cb:
    response = chain.invoke({"topic": "birds"})
    print(response)
    response = chain.invoke({"topic": "fish"})
    print("--")
    print(response)


print()
print("---")
print(f"Total Tokens: {cb.total_tokens}")
print(f"Prompt Tokens: {cb.prompt_tokens}")
print(f"Completion Tokens: {cb.completion_tokens}")
print(f"Total Cost (USD): ${cb.total_cost}")
```
```output


Why did the chicken go to the seance?

To talk to the other side of the road!
--


Why did the fish need a lawyer?

Because it got caught in a net!

---
Total Tokens: 50
Prompt Tokens: 12
Completion Tokens: 38
Total Cost (USD): $9.400000000000001e-05
```

## 流媒体

:::danger

`get_openai_callback` 目前不支持旧版语言模型（例如 `langchain_openai.OpenAI`）的流式令牌计数。如果您想在流式上下文中正确计数令牌，有几个选项：

- 使用聊天模型，如 [本指南](/docs/how_to/chat_token_usage_tracking) 中所述；
- 实现一个 [自定义回调处理程序](/docs/how_to/custom_callbacks/)，使用适当的分词器来计数令牌；
- 使用监控平台，例如 [LangSmith](https://www.langchain.com/langsmith)。
:::

请注意，在流式上下文中使用旧版语言模型时，令牌计数不会更新：

```python
from langchain_community.callbacks import get_openai_callback
from langchain_openai import OpenAI

llm = OpenAI(model_name="gpt-3.5-turbo-instruct")

with get_openai_callback() as cb:
    for chunk in llm.stream("Tell me a joke"):
        print(chunk, end="", flush=True)
    print(result)
    print("---")
print()

print(f"Total Tokens: {cb.total_tokens}")
print(f"Prompt Tokens: {cb.prompt_tokens}")
print(f"Completion Tokens: {cb.completion_tokens}")
print(f"Total Cost (USD): ${cb.total_cost}")
```
```output


Why don't scientists trust atoms?

Because they make up everything!

Why don't scientists trust atoms?

Because they make up everything.
---

Total Tokens: 0
Prompt Tokens: 0
Completion Tokens: 0
Total Cost (USD): $0.0
```