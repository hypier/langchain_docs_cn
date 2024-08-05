---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/wandb_tracking.ipynb
---

# Weights & Biases

本笔记本介绍了如何将您的 LangChain 实验跟踪到一个集中化的权重与偏差仪表板中。要了解更多关于提示工程和回调的信息，请参考此报告，该报告解释了这两者以及您可以期待看到的结果仪表板。

<a href="https://colab.research.google.com/drive/1DXH4beT4HFaRKy_Vm4PoxhXVDRf7Ym8L?usp=sharing" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开"/></a>

[查看报告](https://wandb.ai/a-sh0ts/langchain_callback_demo/reports/Prompt-Engineering-LLMs-with-LangChain-and-W-B--VmlldzozNjk1NTUw#👋-如何在langchain中构建回调以更好地进行提示工程)

**注意**：_`WandbCallbackHandler` 正在被弃用，取而代之的是 `WandbTracer`_。在未来请使用 `WandbTracer`，因为它更灵活并允许更细粒度的日志记录。要了解更多关于 `WandbTracer` 的信息，请参考 [agent_with_wandb_tracing](/docs/integrations/providers/wandb_tracing) 笔记本或使用以下 [colab 笔记本](http://wandb.me/prompts-quickstart)。要了解更多关于权重与偏差提示的信息，请参考以下 [提示文档](https://docs.wandb.ai/guides/prompts)。

```python
%pip install --upgrade --quiet  wandb
%pip install --upgrade --quiet  pandas
%pip install --upgrade --quiet  textstat
%pip install --upgrade --quiet  spacy
!python -m spacy download en_core_web_sm
```

```python
import os

os.environ["WANDB_API_KEY"] = ""
# os.environ["OPENAI_API_KEY"] = ""
# os.environ["SERPAPI_API_KEY"] = ""
```

```python
from datetime import datetime

from langchain_community.callbacks import WandbCallbackHandler
from langchain_core.callbacks import StdOutCallbackHandler
from langchain_openai import OpenAI
```

```
记录到权重与偏差的回调处理器。

参数：
    job_type (str): 作业类型。
    project (str): 要记录的项目。
    entity (str): 要记录的实体。
    tags (list): 要记录的标签。
    group (str): 要记录的组。
    name (str): 运行的名称。
    notes (str): 要记录的备注。
    visualize (bool): 是否可视化运行。
    complexity_metrics (bool): 是否记录复杂性指标。
    stream_logs (bool): 是否将回调操作流式传输到 W&B。
```

```
WandbCallbackHandler(...) 的默认值

visualize: bool = False,
complexity_metrics: bool = False,
stream_logs: bool = False,
```

注意：对于测试工作流，我们基于 textstat 进行了默认分析，基于 spacy 进行了可视化。

```python
"""主函数。

此函数用于尝试回调处理器。
场景：
1. OpenAI LLM
2. 包含多个子链的链，进行多次生成
3. 带工具的代理
"""
session_group = datetime.now().strftime("%m.%d.%Y_%H.%M.%S")
wandb_callback = WandbCallbackHandler(
    job_type="inference",
    project="langchain_callback_demo",
    group=f"minimal_{session_group}",
    name="llm",
    tags=["test"],
)
callbacks = [StdOutCallbackHandler(), wandb_callback]
llm = OpenAI(temperature=0, callbacks=callbacks)
```


```
# WandbCallbackHandler.flush_tracker(...) 的默认值

reset: bool = True,
finish: bool = False,
```

`flush_tracker` 函数用于将 LangChain 会话记录到权重与偏差中。它接受 LangChain 模块或代理，并至少记录提示和生成，以及 LangChain 模块的序列化形式到指定的权重与偏差项目中。默认情况下，我们重置会话，而不是直接结束会话。

```python
# 场景 1 - LLM
llm_result = llm.generate(["告诉我一个笑话", "给我一首诗"] * 3)
wandb_callback.flush_tracker(llm, name="simple_sequential")
```



```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
```

```python
# 场景 2 - 链
template = """您是一位剧作家。给定剧本的标题，您的工作是为该标题写一个概要。
标题: {title}
剧作家: 这是上述剧本的概要:"""
prompt_template = PromptTemplate(input_variables=["title"], template=template)
synopsis_chain = LLMChain(llm=llm, prompt=prompt_template, callbacks=callbacks)

test_prompts = [
    {
        "title": "关于推动游戏设计边界的优秀视频游戏的纪录片"
    },
    {"title": "可卡因熊 vs 海洛因狼"},
    {"title": "最佳的 MLOps 工具"},
]
synopsis_chain.apply(test_prompts)
wandb_callback.flush_tracker(synopsis_chain, name="agent")
```



```python
from langchain.agents import AgentType, initialize_agent, load_tools
```

```python
# 场景 3 - 带工具的代理
tools = load_tools(["serpapi", "llm-math"], llm=llm)
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
)
agent.run(
    "谁是莱昂纳多·迪卡普里奥的女友？她当前的年龄的 0.43 次方是多少？",
    callbacks=callbacks,
)
wandb_callback.flush_tracker(agent, reset=False, finish=True)
```
