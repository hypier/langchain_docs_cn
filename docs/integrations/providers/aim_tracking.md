---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/aim_tracking.ipynb
---

# 目标

Aim 使可视化和调试 LangChain 执行变得非常简单。Aim 跟踪 LLM 和工具的输入和输出，以及代理的操作。

使用 Aim，您可以轻松调试和检查单个执行：

![](https://user-images.githubusercontent.com/13848158/227784778-06b806c7-74a1-4d15-ab85-9ece09b458aa.png)

此外，您还可以选择并排比较多个执行：

![](https://user-images.githubusercontent.com/13848158/227784994-699b24b7-e69b-48f9-9ffa-e6a6142fd719.png)

Aim 是完全开源的，[了解更多](https://github.com/aimhubio/aim)关于 Aim 的信息。

让我们继续，看看如何启用和配置 Aim 回调。

<h3>使用 Aim 跟踪 LangChain 执行</h3>

在这个笔记本中，我们将探索三种使用场景。首先，我们将安装必要的包并导入某些模块。随后，我们将配置两个环境变量，这些变量可以在 Python 脚本内或通过终端建立。

```python
%pip install --upgrade --quiet  aim
%pip install --upgrade --quiet  langchain
%pip install --upgrade --quiet  langchain-openai
%pip install --upgrade --quiet  google-search-results
```

```python
import os
from datetime import datetime

from langchain_community.callbacks import AimCallbackHandler
from langchain_core.callbacks import StdOutCallbackHandler
from langchain_openai import OpenAI
```

我们的示例使用 GPT 模型作为 LLM，OpenAI 为此提供 API。您可以从以下链接获取密钥：https://platform.openai.com/account/api-keys 。

我们将使用 SerpApi 从 Google 检索搜索结果。要获取 SerpApi 密钥，请访问 https://serpapi.com/manage-api-key 。

```python
os.environ["OPENAI_API_KEY"] = "..."
os.environ["SERPAPI_API_KEY"] = "..."
```

`AimCallbackHandler` 的事件方法接受 LangChain 模块或代理作为输入，并记录至少提示和生成的结果，以及 LangChain 模块的序列化版本，以便记录到指定的 Aim 运行中。

```python
session_group = datetime.now().strftime("%m.%d.%Y_%H.%M.%S")
aim_callback = AimCallbackHandler(
    repo=".",
    experiment_name="场景 1：OpenAI LLM",
)

callbacks = [StdOutCallbackHandler(), aim_callback]
llm = OpenAI(temperature=0, callbacks=callbacks)
```

`flush_tracker` 函数用于在 Aim 上记录 LangChain 资产。默认情况下，会议会被重置，而不是完全终止。

<h3>场景 1</h3> 在第一个场景中，我们将使用 OpenAI LLM。

```python
# 场景 1 - LLM
llm_result = llm.generate(["告诉我一个笑话", "给我讲一首诗"] * 3)
aim_callback.flush_tracker(
    langchain_asset=llm,
    experiment_name="场景 2：多个子链的链与多个生成",
)
```

<h3>场景 2</h3> 场景二涉及多个生成中多个子链的链。

```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
```

```python
# 场景 2 - 链
template = """你是一位剧作家。根据剧本的标题，你的工作是为该标题撰写概要。
标题：{title}
剧作家：这是上述剧本的概要："""
prompt_template = PromptTemplate(input_variables=["title"], template=template)
synopsis_chain = LLMChain(llm=llm, prompt=prompt_template, callbacks=callbacks)

test_prompts = [
    {
        "title": "关于推动游戏设计边界的优秀视频游戏的纪录片"
    },
    {"title": "猎豹惊人速度背后的现象"},
    {"title": "一流的 MLOps 工具"},
]
synopsis_chain.apply(test_prompts)
aim_callback.flush_tracker(
    langchain_asset=synopsis_chain, experiment_name="场景 3：带工具的代理"
)
```

<h3>场景 3</h3> 第三个场景涉及带工具的代理。

```python
from langchain.agents import AgentType, initialize_agent, load_tools
```

```python
# 场景 3 - 带工具的代理
tools = load_tools(["serpapi", "llm-math"], llm=llm, callbacks=callbacks)
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    callbacks=callbacks,
)
agent.run(
    "莱昂纳多·迪卡普里奥的女朋友是谁？她目前的年龄的 0.43 次方是多少？"
)
aim_callback.flush_tracker(langchain_asset=agent, reset=False, finish=True)
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m 我需要找出莱昂纳多·迪卡普里奥的女朋友是谁，然后计算她的年龄的 0.43 次方。
行动：搜索
行动输入：“莱昂纳多·迪卡普里奥女朋友”[0m
观察：[36;1m[1;3m莱昂纳多·迪卡普里奥似乎在与女友卡米拉·莫罗内分手后证实了他长期以来的爱情生活理论...[0m
思考：[32;1m[1;3m 我需要找出卡米拉·莫罗内的年龄
行动：搜索
行动输入：“卡米拉·莫罗内年龄”[0m
观察：[36;1m[1;3m25岁[0m
思考：[32;1m[1;3m 我需要计算 25 的 0.43 次方
行动：计算器
行动输入：25^0.43[0m
观察：[33;1m[1;3m答案：3.991298452658078
[0m
思考：[32;1m[1;3m 我现在知道最终答案了
最终答案：卡米拉·莫罗内是莱昂纳多·迪卡普里奥的女朋友，她目前的年龄的 0.43 次方是 3.991298452658078。[0m

[1m> 完成链。[0m
```