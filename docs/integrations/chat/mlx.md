---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/mlx.ipynb
---

# MLX

本笔记本展示了如何开始使用 `MLX` LLM 作为聊天模型。

特别地，我们将：
1. 利用 [MLXPipeline](https://github.com/langchain-ai/langchain/blob/master/libs/community/langchain_community/llms/mlx_pipeline.py)，
2. 利用 `ChatMLX` 类使这些 LLM 能够与 LangChain 的 [聊天消息](https://python.langchain.com/docs/modules/model_io/chat/#messages) 抽象接口交互。
3. 演示如何使用开源 LLM 来驱动 `ChatAgent` 管道。



```python
%pip install --upgrade --quiet  mlx-lm transformers huggingface_hub
```

## 1. 实例化 LLM

有三个 LLM 选项可供选择。


```python
from langchain_community.llms.mlx_pipeline import MLXPipeline

llm = MLXPipeline.from_model_id(
    "mlx-community/quantized-gemma-2b-it",
    pipeline_kwargs={"max_tokens": 10, "temp": 0.1},
)
```

## 2. 实例化 `ChatMLX` 以应用聊天模板

实例化聊天模型和一些要传递的消息。

```python
from langchain_community.chat_models.mlx import ChatMLX
from langchain_core.messages import HumanMessage

messages = [
    HumanMessage(
        content="What happens when an unstoppable force meets an immovable object?"
    ),
]

chat_model = ChatMLX(llm=llm)
```

检查聊天消息在 LLM 调用中的格式。

```python
chat_model._to_chat_prompt(messages)
```

调用模型。

```python
res = chat_model.invoke(messages)
print(res.content)
```

## 3. 作为代理进行测试！

在这里，我们将测试 `gemma-2b-it` 作为一个零-shot `ReAct` 代理。下面的例子取自 [这里](https://python.langchain.com/docs/modules/agents/agent_types/react#using-chat-models)。

> 注意：要运行此部分，您需要将 [SerpAPI Token](https://serpapi.com/) 保存为环境变量：`SERPAPI_API_KEY`


```python
from langchain import hub
from langchain.agents import AgentExecutor, load_tools
from langchain.agents.format_scratchpad import format_log_to_str
from langchain.agents.output_parsers import (
    ReActJsonSingleInputOutputParser,
)
from langchain.tools.render import render_text_description
from langchain_community.utilities import SerpAPIWrapper
```

使用 `react-json` 风格的提示和访问搜索引擎及计算器来配置代理。


```python
# setup tools
tools = load_tools(["serpapi", "llm-math"], llm=llm)

# setup ReAct style prompt
prompt = hub.pull("hwchase17/react-json")
prompt = prompt.partial(
    tools=render_text_description(tools),
    tool_names=", ".join([t.name for t in tools]),
)

# define the agent
chat_model_with_stop = chat_model.bind(stop=["\nObservation"])
agent = (
    {
        "input": lambda x: x["input"],
        "agent_scratchpad": lambda x: format_log_to_str(x["intermediate_steps"]),
    }
    | prompt
    | chat_model_with_stop
    | ReActJsonSingleInputOutputParser()
)

# instantiate AgentExecutor
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```


```python
agent_executor.invoke(
    {
        "input": "Who is Leo DiCaprio's girlfriend? What is her current age raised to the 0.43 power?"
    }
)
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)