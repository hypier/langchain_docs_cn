---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/bittensor.ipynb
---

# Bittensor

>[Bittensor](https://bittensor.com/) 是一个类似于比特币的挖矿网络，包含内置激励机制，旨在鼓励矿工贡献计算能力和知识。
>
>`NIBittensorLLM` 由 [Neural Internet](https://neuralinternet.ai/) 开发，基于 `Bittensor` 提供支持。

>该 LLM 展示了去中心化 AI 的真正潜力，通过从 `Bittensor 协议` 中提供最佳响应，包含各种 AI 模型，如 `OpenAI`、`LLaMA2` 等。

用户可以在 [Validator Endpoint Frontend](https://api.neuralinternet.ai/) 查看他们的日志、请求和 API 密钥。然而，目前禁止更改配置；否则，用户的查询将被阻止。

如果您遇到任何困难或有任何问题，请随时通过 [GitHub](https://github.com/Kunj-2206)、[Discord](https://discordapp.com/users/683542109248159777) 联系我们的开发者，或加入我们的 Discord 服务器以获取最新更新和咨询 [Neural Internet](https://discord.gg/neuralinternet)。

## NIBittensorLLM 的不同参数和响应处理

```python
import json
from pprint import pprint

from langchain.globals import set_debug
from langchain_community.llms import NIBittensorLLM

set_debug(True)

# 在 NIBittensorLLM 中，系统参数是可选的，但您可以设置任何您希望与模型进行的操作
llm_sys = NIBittensorLLM(
    system_prompt="您的任务是根据用户提示确定响应。像我这样解释，就像我是一个项目的技术负责人"
)
sys_resp = llm_sys(
    "什么是 bittensor，去中心化 AI 的潜在好处是什么？"
)
print(f"使用设置的系统提示提供的 LLM 响应是 : {sys_resp}")

# top_responses 参数可以根据其参数值提供多个响应
# 以下代码检索前 10 个矿工的响应，所有响应均为 json 格式

# Json 响应结构是
""" {
    "choices":  [
                    {"index": Bittensor's Metagraph index number,
                    "uid": Unique Identifier of a miner,
                    "responder_hotkey": Hotkey of a miner,
                    "message":{"role":"assistant","content": Contains actual response},
                    "response_ms": Time in millisecond required to fetch response from a miner} 
                ]
    } """

multi_response_llm = NIBittensorLLM(top_responses=10)
multi_resp = multi_response_llm.invoke("什么是神经网络喂养机制？")
json_multi_resp = json.loads(multi_resp)
pprint(json_multi_resp)
```

## 使用 NIBittensorLLM 与 LLMChain 和 PromptTemplate

```python
from langchain.chains import LLMChain
from langchain.globals import set_debug
from langchain_community.llms import NIBittensorLLM
from langchain_core.prompts import PromptTemplate

set_debug(True)

template = """Question: {question}

Answer: Let's think step by step."""


prompt = PromptTemplate.from_template(template)

# NIBittensorLLM 中的系统参数是可选的，但您可以设置任何您想要与模型一起执行的内容
llm = NIBittensorLLM(
    system_prompt="Your task is to determine response based on user prompt."
)

llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "What is bittensor?"

llm_chain.run(question)
```

## 使用 NIBittensorLLM 与对话代理和 Google 搜索工具

```python
from langchain_community.utilities import GoogleSearchAPIWrapper
from langchain_core.tools import Tool

search = GoogleSearchAPIWrapper()

tool = Tool(
    name="Google Search",
    description="Search Google for recent results.",
    func=search.run,
)
```

```python
from langchain import hub
from langchain.agents import (
    AgentExecutor,
    create_react_agent,
)
from langchain.memory import ConversationBufferMemory
from langchain_community.llms import NIBittensorLLM

tools = [tool]

prompt = hub.pull("hwchase17/react")


llm = NIBittensorLLM(
    system_prompt="Your task is to determine a response based on user prompt"
)

memory = ConversationBufferMemory(memory_key="chat_history")

agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, memory=memory)

response = agent_executor.invoke({"input": prompt})
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)