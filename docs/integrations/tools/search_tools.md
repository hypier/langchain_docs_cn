---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/search_tools.ipynb
---
# Search Tools

This notebook shows off usage of various search tools.


```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain_openai import OpenAI
```


```python
llm = OpenAI(temperature=0)
```

## Google Serper API Wrapper

First, let's try to use the Google Serper API tool.


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

Second, let's try SearchApi tool.


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

Now, let's use the SerpAPI tool.


```python
tools = load_tools(["serpapi"], llm=llm)
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
[32;1m[1;3m I need to find out what the current weather is in Pomfret.
Action: Search
Action Input: "weather in Pomfret"[0m
Observation: [36;1m[1;3m{'type': 'weather_result', 'temperature': '69', 'unit': 'Fahrenheit', 'precipitation': '2%', 'humidity': '90%', 'wind': '1 mph', 'location': 'Pomfret, CT', 'date': 'Sunday 9:00 PM', 'weather': 'Clear'}[0m
Thought:[32;1m[1;3m I now know the current weather in Pomfret.
Final Answer: The current weather in Pomfret is 69 degrees Fahrenheit, 2% precipitation, 90% humidity, and 1 mph wind. It is currently clear.[0m

[1m> Finished chain.[0m
```


```output
'The current weather in Pomfret is 69 degrees Fahrenheit, 2% precipitation, 90% humidity, and 1 mph wind. It is currently clear.'
```


## GoogleSearchAPIWrapper

Now, let's use the official Google Search API Wrapper.


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
[32;1m[1;3m I should look up the current weather conditions.
Action: Google Search
Action Input: "weather in Pomfret"[0m
Observation: [36;1m[1;3mShowers early becoming a steady light rain later in the day. Near record high temperatures. High around 60F. Winds SW at 10 to 15 mph. Chance of rain 60%. Pomfret, CT Weather Forecast, with current conditions, wind, air quality, and what to expect for the next 3 days. Hourly Weather-Pomfret, CT. As of 12:52 am EST. Special Weather Statement +2 ... Hazardous Weather Conditions. Special Weather Statement ... Pomfret CT. Tonight ... National Digital Forecast Database Maximum Temperature Forecast. Pomfret Center Weather Forecasts. Weather Underground provides local & long-range weather forecasts, weatherreports, maps & tropical weather conditions for ... Pomfret, CT 12 hour by hour weather forecast includes precipitation, temperatures, sky conditions, rain chance, dew-point, relative humidity, wind direction ... North Pomfret Weather Forecasts. Weather Underground provides local & long-range weather forecasts, weatherreports, maps & tropical weather conditions for ... Today's Weather - Pomfret, CT. Dec 31, 2022 4:00 PM. Putnam MS. --. Weather forecast icon. Feels like --. Hi --. Lo --. Pomfret, CT temperature trend for the next 14 Days. Find daytime highs and nighttime lows from TheWeatherNetwork.com. Pomfret, MD Weather Forecast Date: 332 PM EST Wed Dec 28 2022. The area/counties/county of: Charles, including the cites of: St. Charles and Waldorf.[0m
Thought:[32;1m[1;3m I now know the current weather conditions in Pomfret.
Final Answer: Showers early becoming a steady light rain later in the day. Near record high temperatures. High around 60F. Winds SW at 10 to 15 mph. Chance of rain 60%.[0m
[1m> Finished AgentExecutor chain.[0m
```


```output
'Showers early becoming a steady light rain later in the day. Near record high temperatures. High around 60F. Winds SW at 10 to 15 mph. Chance of rain 60%.'
```


## SearxNG Meta Search Engine

Here we will be using a self hosted SearxNG meta search engine.


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
Observation: [36;1m[1;3mMainly cloudy with snow showers around in the morning. High around 40F. Winds NNW at 5 to 10 mph. Chance of snow 40%. Snow accumulations less than one inch.

10 Day Weather - Pomfret, MD As of 1:37 pm EST Today 49°/ 41° 52% Mon 27 | Day 49° 52% SE 14 mph Cloudy with occasional rain showers. High 49F. Winds SE at 10 to 20 mph. Chance of rain 50%....

10 Day Weather - Pomfret, VT As of 3:51 am EST Special Weather Statement Today 39°/ 32° 37% Wed 01 | Day 39° 37% NE 4 mph Cloudy with snow showers developing for the afternoon. High 39F....

Pomfret, CT ; Current Weather. 1:06 AM. 35°F · RealFeel® 32° ; TODAY'S WEATHER FORECAST. 3/3. 44°Hi. RealFeel® 50° ; TONIGHT'S WEATHER FORECAST. 3/3. 32°Lo.

Pomfret, MD Forecast Today Hourly Daily Morning 41° 1% Afternoon 43° 0% Evening 35° 3% Overnight 34° 2% Don't Miss Finally, Here’s Why We Get More Colds and Flu When It’s Cold Coast-To-Coast...

Pomfret, MD Weather Forecast | AccuWeather Current Weather 5:35 PM 35° F RealFeel® 36° RealFeel Shade™ 36° Air Quality Excellent Wind E 3 mph Wind Gusts 5 mph Cloudy More Details WinterCast...

Pomfret, VT Weather Forecast | AccuWeather Current Weather 11:21 AM 23° F RealFeel® 27° RealFeel Shade™ 25° Air Quality Fair Wind ESE 3 mph Wind Gusts 7 mph Cloudy More Details WinterCast...

Pomfret Center, CT Weather Forecast | AccuWeather Daily Current Weather 6:50 PM 39° F RealFeel® 36° Air Quality Fair Wind NW 6 mph Wind Gusts 16 mph Mostly clear More Details WinterCast...

12:00 pm · Feels Like36° · WindN 5 mph · Humidity43% · UV Index3 of 10 · Cloud Cover65% · Rain Amount0 in ...

Pomfret Center, CT Weather Conditions | Weather Underground star Popular Cities San Francisco, CA 49 °F Clear Manhattan, NY 37 °F Fair Schiller Park, IL (60176) warning39 °F Mostly Cloudy...[0m
Thought:[32;1m[1;3m I now know the final answer
Final Answer: The current weather in Pomfret is mainly cloudy with snow showers around in the morning. The temperature is around 40F with winds NNW at 5 to 10 mph. Chance of snow is 40%.[0m

[1m> Finished chain.[0m
```


```output
'The current weather in Pomfret is mainly cloudy with snow showers around in the morning. The temperature is around 40F with winds NNW at 5 to 10 mph. Chance of snow is 40%.'
```



## Related

- Tool [conceptual guide](/docs/concepts/#tools)
- Tool [how-to guides](/docs/how_to/#tools)
