---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/wandb_tracing.ipynb
---

# WandB 跟踪

有两种推荐的方法来跟踪您的 LangChains：

1. 将 `LANGCHAIN_WANDB_TRACING` 环境变量设置为 "true"。
1. 使用带有 tracing_enabled() 的上下文管理器来跟踪特定代码块。

**注意** 如果设置了环境变量，则所有代码都会被跟踪，无论它是否在上下文管理器内。

```python
import os

from langchain_community.callbacks import wandb_tracing_enabled

os.environ["LANGCHAIN_WANDB_TRACING"] = "true"

# wandb 文档以使用环境变量配置 wandb
# https://docs.wandb.ai/guides/track/advanced/environment-variables
# 在这里我们配置 wandb 项目名称
os.environ["WANDB_PROJECT"] = "langchain-tracing"

from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
```


```python
# 带有跟踪的代理运行。确保适当地设置 OPENAI_API_KEY 以运行此示例。

llm = OpenAI(temperature=0)
tools = load_tools(["llm-math"], llm=llm)
```


```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)

agent.run("2 的 .123243 次方是多少？")  # 这应该被跟踪
# 控制台中应该打印出类似以下的跟踪会话的 URL：
# https://wandb.ai/<wandb_entity>/<wandb_project>/runs/<run_id>
# 此 URL 可用于在 wandb 中查看跟踪会话。
```


```python
# 现在，我们取消设置环境变量并使用上下文管理器。
if "LANGCHAIN_WANDB_TRACING" in os.environ:
    del os.environ["LANGCHAIN_WANDB_TRACING"]

# 使用上下文管理器启用跟踪
with wandb_tracing_enabled():
    agent.run("5 的 .123243 次方是多少？")  # 这应该被跟踪

agent.run("2 的 .123243 次方是多少？")  # 这不应该被跟踪
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m 我需要使用计算器来解决这个问题。
Action: Calculator
Action Input: 5^.123243[0m
Observation: [36;1m[1;3m答案: 1.2193914912400514[0m
Thought:[32;1m[1;3m 我现在知道最终答案了。
Final Answer: 1.2193914912400514[0m

[1m> Finished chain.[0m


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m 我需要使用计算器来解决这个问题。
Action: Calculator
Action Input: 2^.123243[0m
Observation: [36;1m[1;3m答案: 1.0891804557407723[0m
Thought:[32;1m[1;3m 我现在知道最终答案了。
Final Answer: 1.0891804557407723[0m

[1m> Finished chain.[0m
```


```output
'1.0891804557407723'
```