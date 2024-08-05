---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/openweathermap.ipynb
---

# OpenWeatherMap

本笔记本介绍如何使用 `OpenWeatherMap` 组件获取天气信息。

首先，您需要注册一个 `OpenWeatherMap API` 密钥：

1. 前往 OpenWeatherMap 并在 [这里](https://openweathermap.org/api/) 注册一个 API 密钥
2. pip install pyowm

接下来，我们需要设置一些环境变量：
1. 将您的 API 密钥保存到 OPENWEATHERMAP_API_KEY 环境变量中

## 使用包装器


```python
import os

from langchain_community.utilities import OpenWeatherMapAPIWrapper

os.environ["OPENWEATHERMAP_API_KEY"] = ""

weather = OpenWeatherMapAPIWrapper()
```


```python
weather_data = weather.run("London,GB")
print(weather_data)
```
```output
在伦敦，GB，当前天气如下：
详细状态：破碎云
风速：2.57 m/s，方向：240°
湿度：55%
温度： 
  - 当前：20.12°C
  - 最高：21.75°C
  - 最低：18.68°C
  - 体感温度：19.62°C
降雨：{}
热指数：无
云层覆盖：75%
```

## 使用工具


```python
import os

from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI

os.environ["OPENAI_API_KEY"] = ""
os.environ["OPENWEATHERMAP_API_KEY"] = ""

llm = OpenAI(temperature=0)

tools = load_tools(["openweathermap-api"], llm)

agent_chain = initialize_agent(
    tools=tools, llm=llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```


```python
agent_chain.run("伦敦的天气怎么样？")
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m I need to find out the current weather in London.
Action: OpenWeatherMap
Action Input: London,GB[0m
Observation: [36;1m[1;3mIn London,GB, the current weather is as follows:
Detailed status: broken clouds
Wind speed: 2.57 m/s, direction: 240°
Humidity: 56%
Temperature: 
  - Current: 20.11°C
  - High: 21.75°C
  - Low: 18.68°C
  - Feels like: 19.64°C
Rain: {}
Heat index: None
Cloud cover: 75%[0m
Thought:[32;1m[1;3m I now know the current weather in London.
Final Answer: The current weather in London is broken clouds, with a wind speed of 2.57 m/s, direction 240°, humidity of 56%, temperature of 20.11°C, high of 21.75°C, low of 18.68°C, and a heat index of None.[0m

[1m> Finished chain.[0m
```


```output
'伦敦的当前天气是多云，风速为2.57 m/s，方向为240°，湿度为56%，温度为20.11°C，最高温度为21.75°C，最低温度为18.68°C，热指数为无。'
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)