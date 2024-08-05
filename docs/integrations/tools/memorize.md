---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/memorize.ipynb
---

# 记忆

通过无监督学习对LLM进行微调以记忆信息。

该工具需要支持微调的LLM。目前，仅支持 `langchain.llms import GradientLLM`。

## 导入


```python
import os

from langchain.agents import AgentExecutor, AgentType, initialize_agent, load_tools
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory
from langchain_community.llms import GradientLLM
```

## 设置环境 API 密钥
确保从 Gradient AI 获取您的 API 密钥。您将获得 $10 的免费信用额度以测试和微调不同的模型。

```python
from getpass import getpass

if not os.environ.get("GRADIENT_ACCESS_TOKEN", None):
    # Access token under https://auth.gradient.ai/select-workspace
    os.environ["GRADIENT_ACCESS_TOKEN"] = getpass("gradient.ai access token:")
if not os.environ.get("GRADIENT_WORKSPACE_ID", None):
    # `ID` listed in `$ gradient workspace list`
    # also displayed after login at at https://auth.gradient.ai/select-workspace
    os.environ["GRADIENT_WORKSPACE_ID"] = getpass("gradient.ai workspace id:")
if not os.environ.get("GRADIENT_MODEL_ADAPTER_ID", None):
    # `ID` listed in `$ gradient model list --workspace-id "$GRADIENT_WORKSPACE_ID"`
    os.environ["GRADIENT_MODEL_ID"] = getpass("gradient.ai model id:")
```

可选：验证您的环境变量 ```GRADIENT_ACCESS_TOKEN``` 和 ```GRADIENT_WORKSPACE_ID``` 以获取当前部署的模型。

## 创建 `GradientLLM` 实例
您可以指定不同的参数，例如模型名称、生成的最大令牌数、温度等。

```python
llm = GradientLLM(
    model_id=os.environ["GRADIENT_MODEL_ID"],
    # # optional: set new credentials, they default to environment variables
    # gradient_workspace_id=os.environ["GRADIENT_WORKSPACE_ID"],
    # gradient_access_token=os.environ["GRADIENT_ACCESS_TOKEN"],
)
```

## 加载工具


```python
tools = load_tools(["memorize"], llm=llm)
```

## 初始化代理


```python
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    # memory=ConversationBufferMemory(memory_key="chat_history", return_messages=True),
)
```

## 运行代理
请代理记忆一段文本。

```python
agent.run(
    "Please remember the fact in detail:\nWith astonishing dexterity, Zara Tubikova set a world record by solving a 4x4 Rubik's Cube variation blindfolded in under 20 seconds, employing only their feet."
)
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m我应该记住这个事实。
动作：记忆
动作输入：Zara T[0m
观察：[36;1m[1;3m训练完成。损失：1.6853971333333335[0m
思考：[32;1m[1;3m我现在知道最终答案。
最终答案：Zara Tubikova 创造了一个世界[0m

[1m> 完成链。[0m
```


```output
'Zara Tubikova 创造了一个世界'
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)