---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/callbacks/comet_tracing.ipynb
---

# 彗星追踪

有两种方法可以使用彗星追踪您的 LangChains 执行：

1. 将 `LANGCHAIN_COMET_TRACING` 环境变量设置为 "true"。这是推荐的方式。
2. 手动导入 `CometTracer` 并显式传递它。


```python
import os

import comet_llm
from langchain_openai import OpenAI

os.environ["LANGCHAIN_COMET_TRACING"] = "true"

# 如果未设置 API 密钥，则连接到 Comet
comet_llm.init()

# 使用环境变量配置彗星的文档
# https://www.comet.com/docs/v2/api-and-sdk/llm-sdk/configuration/
# 在这里我们配置彗星项目
os.environ["COMET_PROJECT_NAME"] = "comet-example-langchain-tracing"

from langchain.agents import AgentType, initialize_agent, load_tools
```


```python
# 带追踪的代理运行。确保正确设置 OPENAI_API_KEY 以运行此示例。

llm = OpenAI(temperature=0)
tools = load_tools(["llm-math"], llm=llm)
```


```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)

agent.run("What is 2 raised to .123243 power?")  # 这应该被追踪
# 控制台中应打印类似以下的链的 URL：
# https://www.comet.com/<workspace>/<project_name>
# 此 URL 可用于在彗星中查看 LLM 链。
```


```python
# 现在，我们取消设置环境变量并使用上下文管理器。
if "LANGCHAIN_COMET_TRACING" in os.environ:
    del os.environ["LANGCHAIN_COMET_TRACING"]

from langchain_community.callbacks.tracers.comet import CometTracer

tracer = CometTracer()

# 重新创建 LLM、工具和代理，并将回调传递给它们
llm = OpenAI(temperature=0)
tools = load_tools(["llm-math"], llm=llm)
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)

agent.run(
    "What is 2 raised to .123243 power?", callbacks=[tracer]
)  # 这应该被追踪
```