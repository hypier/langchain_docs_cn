---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/bing_search.ipynb
---

# Bing搜索

> [Bing搜索](https://learn.microsoft.com/en-us/bing/search-apis/bing-web-search/) 是一项Azure服务，提供安全、无广告、基于位置的搜索结果，从数十亿个网页文档中提取相关信息。通过利用Bing一次API调用的能力，帮助您的用户在全球网络中找到他们所寻找的内容，涵盖数十亿个网页、图像、视频和新闻。

## 设置
按照 [说明](https://learn.microsoft.com/en-us/bing/search-apis/bing-web-search/create-bing-search-service-resource) 创建 Azure Bing Search v7 服务，并获取订阅密钥

集成位于 `langchain-community` 包中。

```python
%pip install -U langchain-community
```

```python
import getpass
import os

os.environ["BING_SUBSCRIPTION_KEY"] = getpass.getpass()
os.environ["BING_SEARCH_URL"] = "https://api.bing.microsoft.com/v7.0/search"
```

```python
from langchain_community.utilities import BingSearchAPIWrapper
```

```python
search = BingSearchAPIWrapper(k=4)
```

```python
search.run("python")
```

```output
'<b>Python is a</b> versatile and powerful language that lets you work quickly and integrate systems more effectively. Learn how to get started, download the latest version, access documentation, find jobs, and join the Python community. <b>Python is a</b> popular programming language for various purposes. Find the latest version of Python for different operating systems, download release notes, and learn about the development process. Learn <b>Python,</b> a popular programming language for web applications, with examples, exercises, and references. Get certified by completing the PYTHON <b>course</b> at W3Schools. Learn the basic concepts and features of <b>Python,</b> a powerful and easy to learn programming language. The tutorial covers topics such as data structures, modules, classes, exceptions, input and output, and more. Learn why and how to use <b>Python,</b> a popular and easy-to-learn programming language. Find installation guides, tutorials, documentation, resources and FAQs for beginners and experienced programmers. Learn about <b>Python,</b> a high-level, general-purpose programming language with a focus on code readability and multiple paradigms. Find out its history, design, features, libraries, implementations, popularity, uses, and influences. Real <b>Python</b> offers tutorials, books, courses, and news for <b>Python</b> developers of all skill levels. Whether you want to learn <b>Python</b> basics, web development, data science, or machine learning, you can find useful articles and code examples here. Learn how to install, use, and extend <b>Python</b> 3.12.3, a popular programming language. Find tutorials, library references, API guides, FAQs, and more. <b>Python</b> is a powerful, fast, friendly and open-source language that runs everywhere. Learn how to get started, explore applications, join the community and access the latest news and events. Learn the basics of <b>Python</b> programming language with examples of numbers, text, variables, and operators. This tutorial covers the syntax, types, and features of <b>Python</b> for beginners.'
```

## 结果数量
您可以使用 `k` 参数来设置结果数量


```python
search = BingSearchAPIWrapper(k=1)
```


```python
search.run("python")
```



```output
'<b>Python</b> is a versatile and powerful language that lets you work quickly and integrate systems more effectively. Learn how to get started, download the latest version, access documentation, find jobs, and join the Python community.'
```

## 元数据结果

通过 BingSearch 运行查询并返回摘要、标题和链接元数据。

- 摘要：结果的描述。
- 标题：结果的标题。
- 链接：结果的链接。


```python
search = BingSearchAPIWrapper()
```


```python
search.results("apples", 5)
```



```output
[{'snippet': '了解<b>苹果</b>的营养成分、抗氧化剂和潜在健康影响。发现<b>苹果</b>如何帮助减肥、糖尿病、心脏病和癌症。',
  'title': '苹果 101：营养事实和健康益处',
  'link': 'https://www.healthline.com/nutrition/foods/apples'},
 {'snippet': '了解<b>苹果</b>如何通过其纤维、抗氧化剂和植物化学物质改善您的健康。找出不同用途的最佳<b>苹果</b>类型、如何购买和储存它们，以及需要注意的副作用。',
  'title': '苹果：营养和健康益处 - WebMD',
  'link': 'https://www.webmd.com/food-recipes/benefits-apples'},
 {'snippet': '<b>苹果</b>是营养丰富、饱腹且多用途的水果，可能降低各种疾病的风险。了解<b>苹果</b>如何通过科学证据支持您的减肥、心脏健康、肠道健康和大脑健康。',
  'title': '苹果的 10 种显著健康益处',
  'link': 'https://www.healthline.com/nutrition/10-health-benefits-of-apples'},
 {'snippet': '苹果是一种由苹果树（Malus spp.，其中包括家苹果或园艺苹果；Malus domestica）生产的圆形可食用水果。苹果树在全球范围内栽培，是 Malus 属中种植最广泛的物种。该树起源于中亚，其野生祖先 Malus sieversii 仍然存在。<b>苹果</b>在欧亚地区种植已有数千年，并被引入 ...',
  'title': '苹果 - 维基百科',
  'link': 'https://en.wikipedia.org/wiki/Apple'},
 {'snippet': '了解世界上最受欢迎和多样化的<b>苹果</b>，从琼浆到酒苹果，附带照片和描述。通过这本全面的<b>苹果</b>指南，发现它们的起源、风味、用途和营养益处。',
  'title': '从 A 到 Z 的 29 种苹果（附照片！） - 生活学习',
  'link': 'https://www.liveeatlearn.com/types-of-apples/'}]
```

## 工具使用


```python
import os

from langchain_community.tools.bing_search import BingSearchResults
from langchain_community.utilities import BingSearchAPIWrapper

api_wrapper = BingSearchAPIWrapper()
tool = BingSearchResults(api_wrapper=api_wrapper)
tool
```



```output
BingSearchResults(api_wrapper=BingSearchAPIWrapper(bing_subscription_key='<your subscription key>', bing_search_url='https://api.bing.microsoft.com/v7.0/search', k=10, search_kwargs={}))
```



```python
import json

# .invoke wraps utility.results
response = tool.invoke("上海的天气怎么样？")
response = json.loads(response.replace("'", '"'))
for item in response:
    print(item)
```
```output
{'snippet': '<b>上海</b>, <b>上海</b>, 中国 <b>天气</b> 预报，包含当前天气、风速、空气质量及未来3天的预报。', 'title': '上海，上海，中国天气预报 | AccuWeather', 'link': 'https://www.accuweather.com/en/cn/shanghai/106577/weather-forecast/106577'}
{'snippet': '当前 <b>天气</b> <b>在上海</b> 和今天、明天及未来14天的预报', 'title': '上海，上海市，中国的天气 - timeanddate.com', 'link': 'https://www.timeanddate.com/weather/china/shanghai'}
{'snippet': '<b>上海</b> 14天扩展预报。<b>今天的天气</b> <b>天气</b> 每小时14天预报 昨天/过去的<b>天气</b> 气候（平均）当前：73°F。阵雨。部分多云。(<b>天气</b>站：<b>上海</b>虹桥机场，中国)。查看更多当前的<b>天气</b>。', 'title': '上海，上海市，中国14天天气预报', 'link': 'https://www.timeanddate.com/weather/china/shanghai/ext'}
{'snippet': '<b>上海</b> - <b>天气</b>预警发布14天预报。<b>天气</b>预警发布。预报 - <b>上海</b>。逐日预报。最后更新于今天18:00。今晚，晴朗的天空和微风。晴空。', 'title': '上海 - BBC天气', 'link': 'https://www.bbc.com/weather/1796236'}
```

## 链接

我们在这里展示如何将其用作 [agent](/docs/tutorials/agents) 的一部分。我们使用 OpenAI Functions Agent，因此我们需要设置和安装所需的依赖项。我们还将使用 [LangSmith Hub](https://smith.langchain.com/hub) 来提取提示，因此我们需要安装它。

```python
# you need a model to use in the chain
%pip install --upgrade --quiet langchain langchain-openai langchainhub langchain-community
```

```python
import getpass
import os

from langchain import hub
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_openai import AzureChatOpenAI

os.environ["AZURE_OPENAI_API_KEY"] = getpass.getpass()
os.environ["AZURE_OPENAI_ENDPOINT"] = "https://<your-endpoint>.openai.azure.com/"
os.environ["AZURE_OPENAI_API_VERSION"] = "2023-06-01-preview"
os.environ["AZURE_OPENAI_DEPLOYMENT_NAME"] = "<your-deployment-name>"

instructions = """You are an assistant."""
base_prompt = hub.pull("langchain-ai/openai-functions-template")
prompt = base_prompt.partial(instructions=instructions)
llm = AzureChatOpenAI(
    openai_api_key=os.environ["AZURE_OPENAI_API_KEY"],
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
    azure_deployment=os.environ["AZURE_OPENAI_DEPLOYMENT_NAME"],
    openai_api_version=os.environ["AZURE_OPENAI_API_VERSION"],
)
tool = BingSearchResults(api_wrapper=api_wrapper)
tools = [tool]
agent = create_tool_calling_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
)
agent_executor.invoke({"input": "What happened in the latest burning man floods?"})
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Invoking: `bing_search_results_json` with `{'query': 'latest burning man floods'}`


[0m[36;1m[1;3m[{'snippet': 'Live Updates. Thousands stranded at <b>Burning</b> <b>Man</b> festival after heavy rains. By Maureen Chowdhury, Steve Almasyand Matt Meyer, CNN. Updated 9:00 PM EDT, Sun September 3, 2023. Link Copied!', 'title': 'Thousands stranded at Burning Man festival after heavy rains', 'link': 'https://www.cnn.com/us/live-news/nevada-desert-burning-man-weather-rain-09-03-23/index.html'}, {'snippet': 'Black Rock Forest, where around 70,000 <b>Burning</b> <b>Man</b> attendees are gathered for the festival, is in the northwest. &quot;Flash <b>flooding</b> caused by excessive rainfall&quot; is possible in parts of eastern ...', 'title': 'Burning Man flooding keeps thousands stranded at Nevada site as ...', 'link': 'https://www.nbcnews.com/news/us-news/live-blog/live-updates-burning-man-flooding-keeps-thousands-stranded-nevada-site-rcna103193'}, {'snippet': 'Thousands of <b>Burning</b> <b>Man</b> attendees finally made their mass exodus after intense rain over the weekend flooded camp sites and filled them with thick, ankle-deep mud – stranding more than 70,000 ...', 'title': 'Burning Man attendees make a mass exodus after a dramatic weekend ... - CNN', 'link': 'https://www.cnn.com/2023/09/05/us/burning-man-storms-shelter-exodus-tuesday/index.html'}, {'snippet': 'FILE - In this satellite photo provided by Maxar Technologies, an overview of <b>Burning</b> <b>Man</b> festival in Black Rock, Nev on Monday, Aug. 28, 2023. Authorities in Nevada were investigating a death at the site of the <b>Burning</b> <b>Man</b> festival where thousands of attendees remained stranded as <b>flooding</b> from storms swept through the Nevada desert.', 'title': 'Wait times to exit Burning Man drop after flooding left tens of ...', 'link': 'https://apnews.com/article/burning-man-flooding-nevada-stranded-0726190c9f8378935e2a3cce7f154785'}][0m[32;1m[1;3mIn the latest Burning Man festival, heavy rains caused flooding and resulted in thousands of attendees being stranded. The festival took place in Black Rock Forest, Nevada, and around 70,000 people were gathered for the event. The excessive rainfall led to flash flooding in some parts of the area. As a result, camp sites were filled with ankle-deep mud, making it difficult for people to leave. Authorities were investigating a death at the festival site, which was affected by the flooding. However, in the following days, thousands of Burning Man attendees were able to make a mass exodus after the rain subsided.[0m

[1m> Finished chain.[0m
```
```output
{'input': 'What happened in the latest burning man floods?',
 'output': 'In the latest Burning Man festival, heavy rains caused flooding and resulted in thousands of attendees being stranded. The festival took place in Black Rock Forest, Nevada, and around 70,000 people were gathered for the event. The excessive rainfall led to flash flooding in some parts of the area. As a result, camp sites were filled with ankle-deep mud, making it difficult for people to leave. Authorities were investigating a death at the festival site, which was affected by the flooding. However, in the following days, thousands of Burning Man attendees were able to make a mass exodus after the rain subsided.'}
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)