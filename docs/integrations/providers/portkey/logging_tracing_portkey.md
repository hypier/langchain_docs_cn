---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/portkey/logging_tracing_portkey.ipynb
---

# 日志、追踪和监控

在使用 Langchain 构建应用程序或代理时，您最终会进行多次 API 调用以满足单个用户请求。然而，当您想要分析这些请求时，这些请求并没有被串联起来。通过 [**Portkey**](/docs/integrations/providers/portkey/)，来自单个用户请求的所有嵌入、完成和其他请求将被记录并追踪到一个共同的 ID，从而使您能够全面了解用户互动。

本笔记本作为逐步指南，介绍如何在您的 Langchain 应用中使用 `Portkey` 记录、追踪和监控 Langchain LLM 调用。

首先，让我们导入 Portkey、OpenAI 以及 Agent 工具


```python
import os

from langchain.agents import AgentExecutor, create_openai_tools_agent
from langchain_openai import ChatOpenAI
from portkey_ai import PORTKEY_GATEWAY_URL, createHeaders
```

在下面粘贴您的 OpenAI API 密钥。[(您可以在这里找到它)](https://platform.openai.com/account/api-keys)


```python
os.environ["OPENAI_API_KEY"] = "..."
```

## 获取 Portkey API 密钥
1. 在 [这里注册 Portkey](https://app.portkey.ai/signup)
2. 在您的 [仪表板](https://app.portkey.ai/) 上，点击左下角的个人资料图标，然后点击“复制 API 密钥”
3. 将其粘贴到下面


```python
PORTKEY_API_KEY = "..."  # Paste your Portkey API Key here
```

## 设置追踪 ID
1. 在下面为您的请求设置追踪 ID
2. 对于来自单个请求的所有 API 调用，追踪 ID 可以是相同的


```python
TRACE_ID = "uuid-trace-id"  # Set trace id here
```

## 生成 Portkey 头


```python
portkey_headers = createHeaders(
    api_key=PORTKEY_API_KEY, provider="openai", trace_id=TRACE_ID
)
```

定义提示和要使用的工具


```python
from langchain import hub
from langchain_core.tools import tool

prompt = hub.pull("hwchase17/openai-tools-agent")


@tool
def multiply(first_int: int, second_int: int) -> int:
    """将两个整数相乘。"""
    return first_int * second_int


@tool
def exponentiate(base: int, exponent: int) -> int:
    "将底数指数化。"
    return base**exponent


tools = [multiply, exponentiate]
```

像往常一样运行您的代理。**唯一**的变化是我们现在将**包含上述头**在请求中。


```python
model = ChatOpenAI(
    base_url=PORTKEY_GATEWAY_URL, default_headers=portkey_headers, temperature=0
)

# 构造 OpenAI 工具代理
agent = create_openai_tools_agent(model, tools, prompt)

# 通过传入代理和工具创建代理执行器
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

agent_executor.invoke(
    {
        "input": "将 3 的五次方乘以 36，然后对结果进行平方"
    }
)
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m
调用: `exponentiate` 使用 `{'base': 3, 'exponent': 5}`


[0m[33;1m[1;3m243[0m[32;1m[1;3m
调用: `multiply` 使用 `{'first_int': 243, 'second_int': 36}`


[0m[36;1m[1;3m8748[0m[32;1m[1;3m
调用: `exponentiate` 使用 `{'base': 8748, 'exponent': 2}`


[0m[33;1m[1;3m76527504[0m[32;1m[1;3m将 3 的五次方乘以 36，然后对结果进行平方的结果是 76,527,504。[0m

[1m> 完成链。[0m
```


```output
{'input': '将 3 的五次方乘以 36，然后对结果进行平方',
 'output': '将 3 的五次方乘以 36，然后对结果进行平方的结果是 76,527,504。'}
```

## 如何在 Portkey 上工作日志记录和追踪

**日志记录**
- 通过 Portkey 发送请求确保所有请求默认被记录
- 每个请求日志包含 `timestamp`、`model name`、`total cost`、`request time`、`request json`、`response json` 和其他 Portkey 特性

**[追踪](https://portkey.ai/docs/product/observability-modern-monitoring-for-llms/traces)**
- 追踪 ID 与每个请求一起传递，并在 Portkey 仪表板的日志中可见
- 如果需要，您还可以为每个请求设置一个 **独特的追踪 ID**
- 您还可以将用户反馈附加到追踪 ID 上。[更多信息请点击这里](https://portkey.ai/docs/product/observability-modern-monitoring-for-llms/feedback)

对于上述请求，您将能够查看完整的日志追踪，如下所示
![View Langchain traces on Portkey](https://assets.portkey.ai/docs/agent_tracing.gif)

## 高级 LLMOps 功能 - 缓存、标记、重试

除了日志记录和追踪，Portkey 还提供更多功能，增强您现有工作流程的生产能力：

**缓存**

从缓存中响应之前服务过的客户查询，而不是再次发送到 OpenAI。匹配精确字符串或语义相似字符串。缓存可以节省成本并将延迟减少 20 倍。[文档](https://portkey.ai/docs/product/ai-gateway-streamline-llm-integrations/cache-simple-and-semantic)

**重试**

自动重新处理任何未成功的 API 请求 **`最多 5`** 次。使用 **`指数退避`** 策略，间隔重试尝试以防止网络过载。[文档](https://portkey.ai/docs/product/ai-gateway-streamline-llm-integrations)

**标记**

使用预定义的标签详细跟踪和审计每个用户交互。[文档](https://portkey.ai/docs/product/observability-modern-monitoring-for-llms/metadata)