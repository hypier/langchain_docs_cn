---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/search_tools.ipynb
---

# 搜索工具

本笔记本展示了各种搜索工具的使用方法。


```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
```


```python
llm = OpenAI(temperature=0)
```

## Google Serper API Wrapper

首先，让我们尝试使用 Google Serper API 工具。


```python
tools = load_tools(["google-serper"], llm=llm)
```


```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```


```python
agent.run("What is the weather in Pomfret?")
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m I should look up the current weather conditions.
Action: Search
Action Input: "weather in Pomfret"[0m
Observation: [36;1m[1;3m37°F[0m
Thought:[32;1m[1;3m I now know the current temperature in Pomfret.
Final Answer: The current temperature in Pomfret is 37°F.[0m

[1m> Finished chain.[0m
```


```output
'The current temperature in Pomfret is 37°F.'
```

## SearchApi

第二步，让我们尝试使用 SearchApi 工具。


```python
tools = load_tools(["searchapi"], llm=llm)
```


```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```


```python
agent.run("What is the weather in Pomfret?")
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m I need to find out the current weather in Pomfret.
Action: searchapi
Action Input: "weather in Pomfret"[0m
Observation: [36;1m[1;3mThu 14 | Day ... Some clouds this morning will give way to generally sunny skies for the afternoon. High 73F. Winds NW at 5 to 10 mph.
Hourly Weather-Pomfret, CT · 1 pm. 71°. 0%. Sunny. Feels Like71°. WindNW 9 mph · 2 pm. 72°. 0%. Sunny. Feels Like72°. WindNW 9 mph · 3 pm. 72°. 0%. Sunny. Feels ...
10 Day Weather-Pomfret, VT. As of 4:28 am EDT. Today. 68°/48°. 4%. Thu 14 | Day. 68°. 4%. WNW 10 mph. Some clouds this morning will give way to generally ...
Be prepared with the most accurate 10-day forecast for Pomfret, MD with highs, lows, chance of precipitation from The Weather Channel and Weather.com.
Current Weather. 10:00 PM. 65°F. RealFeel® 67°. Mostly cloudy. LOCAL HURRICANE TRACKER. Category2. Lee. Late Friday Night - Saturday Afternoon.
10 Day Weather-Pomfret, NY. As of 5:09 pm EDT. Tonight. --/55°. 10%. Wed 13 | Night. 55°. 10%. NW 11 mph. Some clouds. Low near 55F.
Pomfret CT. Overnight. Overnight: Patchy fog before 3am, then patchy fog after 4am. Otherwise, mostly. Patchy Fog. Low: 58 °F. Thursday.
Isolated showers. Mostly cloudy, with a high near 76. Calm wind. Chance of precipitation is 20%. Tonight. Mostly Cloudy. Mostly cloudy, with a ...
Partly sunny, with a high near 67. Breezy, with a north wind 18 to 22 mph, with gusts as high as 34 mph. Chance of precipitation is 30%. ... A chance of showers ...
Today's Weather - Pomfret, CT ... Patchy fog. Showers. Lows in the upper 50s. Northwest winds around 5 mph. Chance of rain near 100 percent. ... Sunny. Patchy fog ...[0m
Thought:[32;1m[1;3m I now know the final answer
Final Answer: The current weather in Pomfret is mostly cloudy with a high near 67 and a chance of showers. Winds are from the north at 18 to 22 mph with gusts up to 34 mph.[0m

[1m> Finished chain.[0m
```


```output
'The current weather in Pomfret is mostly cloudy with a high near 67 and a chance of showers. Winds are from the north at 18 to 22 mph with gusts up to 34 mph.'
```

## SerpAPI

现在，让我们使用 SerpAPI 工具。


```python
tools = load_tools(["serpapi"], llm=llm)
```


```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```


```python
agent.run("Pomfret 的天气怎么样？")
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m I need to find out what the current weather is in Pomfret.
Action: Search
Action Input: "weather in Pomfret"[0m
Observation: [36;1m[1;3m{'type': 'weather_result', 'temperature': '69', 'unit': 'Fahrenheit', 'precipitation': '2%', 'humidity': '90%', 'wind': '1 mph', 'location': 'Pomfret, CT', 'date': 'Sunday 9:00 PM', 'weather': 'Clear'}[0m
Thought:[32;1m[1;3m I now know the current weather in Pomfret.
Final Answer: The current weather in Pomfret is 69 degrees Fahrenheit, 2% precipitation, 90% humidity, and 1 mph wind. It is currently clear.[0m

[1m> Finished chain.[0m
```


```output
'Pomfret 当前的天气是 69 华氏度，降水量 2%，湿度 90%，风速 1 英里每小时。目前是晴天。'
```

## GoogleSearchAPIWrapper

现在，让我们使用官方的 Google Search API Wrapper。


```python
tools = load_tools(["google-search"], llm=llm)
```


```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```


```python
agent.run("What is the weather in Pomfret?")
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m 我应该查找当前的天气情况。
Action: Google Search
Action Input: "weather in Pomfret"[0m
Observation: [36;1m[1;3m早晨有阵雨，随后变为稳定的细雨。接近创纪录的高温。高温约为60华氏度。西南风速10到15英里每小时。降雨概率60%。Pomfret, CT天气预报，包括当前情况、风、空气质量以及未来3天的预期。Pomfret, CT逐小时天气。截止到东部标准时间12:52。特别天气声明 +2 ... 危险天气条件。特别天气声明 ... Pomfret CT。今晚 ... 国家数字预报数据库最大温度预报。Pomfret Center天气预报。Weather Underground提供本地和长期天气预报、天气报告、地图和热带天气情况... Pomfret, CT逐小时天气预报包括降水、温度、天空状况、降雨概率、露点、相对湿度、风向... North Pomfret天气预报。Weather Underground提供本地和长期天气预报、天气报告、地图和热带天气情况... 今日天气 - Pomfret, CT。2022年12月31日下午4:00。Putnam MS. --。天气预报图标。感觉像 --。高 --。低 --。Pomfret, CT未来14天的温度趋势。查找TheWeatherNetwork.com上的白天高温和夜间低温。Pomfret, MD天气预报日期：2022年12月28日下午3:32东部标准时间。该地区/县/县：查尔斯，包括城市：圣查尔斯和沃尔多夫。[0m
Thought:[32;1m[1;3m 我现在知道了Pomfret的当前天气情况。
Final Answer: 早晨有阵雨，随后变为稳定的细雨。接近创纪录的高温。高温约为60华氏度。西南风速10到15英里每小时。降雨概率60%。[0m
[1m> Finished AgentExecutor chain.[0m
```


```output
'早晨有阵雨，随后变为稳定的细雨。接近创纪录的高温。高温约为60华氏度。西南风速10到15英里每小时。降雨概率60%。'
```

## SearxNG 元搜索引擎

在这里，我们将使用自托管的 SearxNG 元搜索引擎。


```python
tools = load_tools(["searx-search"], searx_host="http://localhost:8888", llm=llm)
```


```python
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```


```python
agent.run("What is the weather in Pomfret")
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m I should look up the current weather
Action: SearX Search
Action Input: "weather in Pomfret"[0m
Observation: [36;1m[1;3m主要多云，早上有雪花。气温约为40华氏度。风向北西北，速度为5到10英里每小时。降雪概率为40%。 

10天天气 - Pomfret, MD 截至今天东部标准时间下午1:37 49°/ 41° 52% 周一 27 | 白天 49° 52% 东南风14英里每小时 多云，偶尔有雨。气温49华氏度。风向东南，速度为10到20英里每小时。降雨概率50%.... 

10天天气 - Pomfret, VT 截至今天东部标准时间凌晨3:51 特殊天气声明 今天39°/ 32° 37% 周三 01 | 白天39° 37% 东北风4英里每小时 多云，下午有雪花。气温39华氏度.... 

Pomfret, CT ; 当前天气。凌晨1:06 35°F · 体感温度® 32° ; 今日天气预报。3/3. 最高44°。体感温度® 50° ; 今晚天气预报。3/3. 最低32°。 

Pomfret, MD 预报 今天 每小时 每日 早上41° 1% 下午43° 0% 晚上35° 3% 夜间34° 2% 不要错过 最后，这就是为什么我们在寒冷时更容易感冒和流感的原因 从海岸到海岸... 

Pomfret, MD 天气预报 | AccuWeather 当前天气 下午5:35 35° F 体感温度® 36° 体感温度阴影™ 36° 空气质量优秀 风向东3英里每小时 风速5英里每小时 多云 更多细节 冬季预报... 

Pomfret, VT 天气预报 | AccuWeather 当前天气 上午11:21 23° F 体感温度® 27° 体感温度阴影™ 25° 空气质量一般 风向东南3英里每小时 风速7英里每小时 多云 更多细节 冬季预报... 

Pomfret Center, CT 天气预报 | AccuWeather 每日当前天气 下午6:50 39° F 体感温度® 36° 空气质量一般 风向西北6英里每小时 风速16英里每小时 大部分晴朗 更多细节 冬季预报... 

中午12:00 · 体感温度36° · 风向北5英里每小时 · 湿度43% · 紫外线指数3/10 · 云量65% · 雨量0英寸... 

Pomfret Center, CT 天气状况 | Weather Underground 星级热门城市 旧金山, CA 49 °F 晴朗 纽约, NY 37 °F 一般 施勒公园, IL (60176) 警告39 °F 大部分多云...[0m
Thought:[32;1m[1;3m I now know the final answer
Final Answer: Pomfret 当前天气主要多云，早上有雪花。气温约为40华氏度，风向北西北，速度为5到10英里每小时。降雪概率为40%。[0m

[1m> Finished chain.[0m
```


```output
'Pomfret 当前天气主要多云，早上有雪花。气温约为40华氏度，风向北西北，速度为5到10英里每小时。降雪概率为40%。'
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)