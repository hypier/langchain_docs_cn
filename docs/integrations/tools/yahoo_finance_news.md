---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/yahoo_finance_news.ipynb
---

# Yahoo Finance 新闻

本笔记本介绍如何使用 `yahoo_finance_news` 工具与代理。

## 设置

首先，您需要安装 `yfinance` Python 包。

```python
%pip install --upgrade --quiet  yfinance
```

## 示例与链

```python
import os

os.environ["OPENAI_API_KEY"] = "..."
```

```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.tools.yahoo_finance_news import YahooFinanceNewsTool
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(temperature=0.0)
tools = [YahooFinanceNewsTool()]
agent_chain = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
)
```

```python
agent_chain.invoke(
    "今天微软股票发生了什么？",
)
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m我应该查看有关微软股票的最新财经新闻。
Action: yahoo_finance_news
Action Input: MSFT[0m
Observation: [36;1m[1;3m微软 (MSFT) 上涨但落后于市场：您应该知道的事
在最新的交易时段，微软 (MSFT) 收于 $328.79，较前一日上涨 +0.12%。[0m
Thought:[32;1m[1;3m我有关于微软股票的最新信息。
Final Answer: 微软 (MSFT) 收于 $328.79，较前一日上涨 +0.12%。[0m

[1m> 完成链。[0m
```

```output
'微软 (MSFT) 收于 $328.79，较前一日上涨 +0.12%。'
```

```python
agent_chain.invoke(
    "今天微软与英伟达相比感觉如何？",
)
```
```output


[1m> 进入新的 AgentExecutor 链...[0m
[32;1m[1;3m我应该比较微软和英伟达当前的情绪。
Action: yahoo_finance_news
Action Input: MSFT[0m
Observation: [36;1m[1;3m微软 (MSFT) 上涨但落后于市场：您应该知道的事
在最新的交易时段，微软 (MSFT) 收于 $328.79，较前一日上涨 +0.12%。[0m
Thought:[32;1m[1;3m我还需要找到英伟达当前的情绪。
Action: yahoo_finance_news
Action Input: NVDA[0m
Observation: [36;1m[1;3m[0m
Thought:[32;1m[1;3m我现在知道了微软和英伟达的当前情绪。
Final Answer: 我无法比较微软和英伟达的情绪，因为我只有关于微软的信息。[0m

[1m> 完成链。[0m
```

```output
'我无法比较微软和英伟达的情绪，因为我只有关于微软的信息。'
```

# YahooFinanceNewsTool 如何工作？

```python
tool = YahooFinanceNewsTool()
```

```python
tool.invoke("NVDA")
```

```output
'没有找到与 NVDA 股票代码相关的公司新闻。'
```

```python
res = tool.invoke("AAPL")
print(res)
```
```output
苹果、博通和卡特彼勒的顶级研究报告
今天的研究日报包含了对 16 只主要股票的新研究报告，包括苹果公司 (AAPL)、博通公司 (AVGO) 和卡特彼勒公司 (CAT)。

苹果股票正面临年度最差月份
根据道琼斯市场数据，苹果 (AAPL) 的股票正面临年度最差月份。 8 月份迄今为止，股票下跌了 4.8%，预计将成为自 2022 年 12 月以来的最差月份，当时下跌了 12%。
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)