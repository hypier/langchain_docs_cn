---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/yahoo_finance_news.ipynb
---
# Yahoo Finance News

This notebook goes over how to use the `yahoo_finance_news` tool with an agent. 


## Setting up

First, you need to install `yfinance` python package.


```python
%pip install --upgrade --quiet  yfinance
```

## Example with Chain


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
    "What happened today with Microsoft stocks?",
)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3mI should check the latest financial news about Microsoft stocks.
Action: yahoo_finance_news
Action Input: MSFT[0m
Observation: [36;1m[1;3mMicrosoft (MSFT) Gains But Lags Market: What You Should Know
In the latest trading session, Microsoft (MSFT) closed at $328.79, marking a +0.12% move from the previous day.[0m
Thought:[32;1m[1;3mI have the latest information on Microsoft stocks.
Final Answer: Microsoft (MSFT) closed at $328.79, with a +0.12% move from the previous day.[0m

[1m> Finished chain.[0m
```


```output
'Microsoft (MSFT) closed at $328.79, with a +0.12% move from the previous day.'
```



```python
agent_chain.invoke(
    "How does Microsoft feels today comparing with Nvidia?",
)
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3mI should compare the current sentiment of Microsoft and Nvidia.
Action: yahoo_finance_news
Action Input: MSFT[0m
Observation: [36;1m[1;3mMicrosoft (MSFT) Gains But Lags Market: What You Should Know
In the latest trading session, Microsoft (MSFT) closed at $328.79, marking a +0.12% move from the previous day.[0m
Thought:[32;1m[1;3mI need to find the current sentiment of Nvidia as well.
Action: yahoo_finance_news
Action Input: NVDA[0m
Observation: [36;1m[1;3m[0m
Thought:[32;1m[1;3mI now know the current sentiment of both Microsoft and Nvidia.
Final Answer: I cannot compare the sentiment of Microsoft and Nvidia as I only have information about Microsoft.[0m

[1m> Finished chain.[0m
```


```output
'I cannot compare the sentiment of Microsoft and Nvidia as I only have information about Microsoft.'
```


# How YahooFinanceNewsTool works?


```python
tool = YahooFinanceNewsTool()
```


```python
tool.invoke("NVDA")
```



```output
'No news found for company that searched with NVDA ticker.'
```



```python
res = tool.invoke("AAPL")
print(res)
```
```output
Top Research Reports for Apple, Broadcom & Caterpillar
Today's Research Daily features new research reports on 16 major stocks, including Apple Inc. (AAPL), Broadcom Inc. (AVGO) and Caterpillar Inc. (CAT).

Apple Stock on Pace for Worst Month of the Year
Apple (AAPL) shares are on pace for their worst month of the year, according to Dow Jones Market Data.  The stock is down 4.8% so far in August, putting it on pace for its worst month since December 2022, when it fell 12%.
```

## Related

- Tool [conceptual guide](/docs/concepts/#tools)
- Tool [how-to guides](/docs/how_to/#tools)
