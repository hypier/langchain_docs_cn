---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/amadeus.ipynb
---

# Amadeus

本笔记将指导您如何将 LangChain 连接到 `Amadeus` 旅行 API。

该 `Amadeus` 工具包允许代理在旅行方面做出决策，特别是在搜索和预订航班方面。

要使用此工具包，您需要准备好您的 Amadeus API 密钥，详细信息请参见 [开始使用 Amadeus 自助服务 API](https://developers.amadeus.com/get-started/get-started-with-self-service-apis-335)。一旦您收到了 AMADEUS_CLIENT_ID 和 AMADEUS_CLIENT_SECRET，您可以将它们作为环境变量输入如下。

注意：Amadeus 自助服务 API 提供了一个具有 [免费有限数据](https://amadeus4dev.github.io/developer-guides/test-data/) 的测试环境。这使开发人员能够在将应用程序部署到生产之前构建和测试它们。要访问实时数据，您需要 [迁移到生产环境](https://amadeus4dev.github.io/developer-guides/API-Keys/moving-to-production/)。


```python
%pip install --upgrade --quiet  amadeus > /dev/null
```


```python
%pip install -qU langchain-community
```

## 设置环境变量

工具包将读取 AMADEUS_CLIENT_ID 和 AMADEUS_CLIENT_SECRET 环境变量以验证用户，因此您需要在此设置它们。

```python
# Set environmental variables here
import os

os.environ["AMADEUS_CLIENT_ID"] = "CLIENT_ID"
os.environ["AMADEUS_CLIENT_SECRET"] = "CLIENT_SECRET"

# os.environ["AMADEUS_HOSTNAME"] = "production" or "test"
```

## 创建 Amadeus 工具包并获取工具

首先，您需要创建工具包，以便稍后可以访问其工具。

默认情况下，`AmadeusToolkit` 使用 `ChatOpenAI` 来识别离给定位置最近的机场。要使用它，只需设置 `OPENAI_API_KEY`。



```python
os.environ["OPENAI_API_KEY"] = "YOUR_OPENAI_API_KEY"
```


```python
from langchain_community.agent_toolkits.amadeus.toolkit import AmadeusToolkit

toolkit = AmadeusToolkit()
tools = toolkit.get_tools()
```

或者，您可以使用 langchain 支持的任何 LLM，例如 `HuggingFaceHub`。 


```python
from langchain_community.llms import HuggingFaceHub

os.environ["HUGGINGFACEHUB_API_TOKEN"] = "YOUR_HF_API_TOKEN"

llm = HuggingFaceHub(
    repo_id="tiiuae/falcon-7b-instruct",
    model_kwargs={"temperature": 0.5, "max_length": 64},
)

toolkit_hf = AmadeusToolkit(llm=llm)
```

"departureDateTimeEarliest": "2024-03-10T00:00:00"
  }
}
```[0m[33;1m[1;3m[{'price': {'total': '593.35', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T10:54:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T13:15:00'}, 'flightNumber': '1634', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '652.68', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T07:25:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T09:46:00'}, 'flightNumber': '380', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-10T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-10T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '765.35', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T16:42:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T17:49:00'}, 'flightNumber': '2655', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-11T17:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T20:07:00'}, 'flightNumber': '4910', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '810.82', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T12:35:00'}, 'arrival': {'iataCode': 'IAD', 'at': '2024-03-10T16:20:00'}, 'flightNumber': '358', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-11T07:45:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T08:51:00'}, 'flightNumber': '4645', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '810.82', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T12:35:00'}, 'arrival': {'iataCode': 'IAD', 'at': '2024-03-10T16:20:00'}, 'flightNumber': '358', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-11T09:45:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T10:42:00'}, 'flightNumber': '5215', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '815.99', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T12:35:00'}, 'arrival': {'iataCode': 'IAD', 'at': '2024-03-10T16:20:00'}, 'flightNumber': '358', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'IAD', 'at': '2024-03-11T07:00:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-11T08:03:00'}, 'flightNumber': '418', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '901.12', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:00:00'}, 'arrival': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T10:25:00'}, 'flightNumber': '1517', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T13:15:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T14:50:00'}, 'flightNumber': '323', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-10T19:35:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-10T21:19:00'}, 'flightNumber': '5413', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '901.12', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:00:00'}, 'arrival': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T10:25:00'}, 'flightNumber': '1517', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T11:40:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T13:23:00'}, 'flightNumber': '1027', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-10T19:35:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-10T21:19:00'}, 'flightNumber': '5413', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '919.40', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:00:00'}, 'arrival': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T10:25:00'}, 'flightNumber': '1517', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T15:10:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T16:57:00'}, 'flightNumber': '1504', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-10T19:35:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-10T21:19:00'}, 'flightNumber': '5413', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '963.36', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T13:45:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T14:45:00'}, 'flightNumber': '1380', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T17:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-10T20:07:00'}, 'flightNumber': '4910', 'carrier': 'UNITED AIRLINES'}]}][0m[32;1m[1;3mObservation: The earliest flight on March 10, 2024, from Dallas, Texas (DFW) to Lincoln, Nebraska (LNK) lands at 2:19 PM.
Final Answer: The earliest flight on March 10, 2024, leaving Dallas, Texas to Lincoln, Nebraska lands in Nebraska at 2:19 PM.[0m

[1m> Finished chain.[0m
```


```output
{'input': 'At what time does earliest flight on March 10, 2024 leaving Dallas, Texas to Lincoln, Nebraska land in Nebraska?',
 'output': 'The earliest flight on March 10, 2024, leaving Dallas, Texas to Lincoln, Nebraska lands in Nebraska at 2:19 PM.'}
```

```json
"departureDateTimeEarliest": "2024-03-10T00:00:00",
    "departureDateTimeLatest": "2024-03-10T23:59:59"
  }
}
```
[0m[33;1m[1;3m[{'price': {'total': '593.35', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T14:20:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T16:49:00'}, 'flightNumber': '1583', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '593.35', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T10:54:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T13:15:00'}, 'flightNumber': '1634', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '652.68', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T07:25:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-10T09:46:00'}, 'flightNumber': '380', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-10T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-10T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '666.77', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T12:35:00'}, 'arrival': {'iataCode': 'IAD', 'at': '2024-03-10T16:20:00'}, 'flightNumber': '358', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'IAD', 'at': '2024-03-11T08:35:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '1', 'at': '2024-03-11T09:39:00'}, 'flightNumber': '1744', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '666.77', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T12:35:00'}, 'arrival': {'iataCode': 'IAD', 'at': '2024-03-10T16:20:00'}, 'flightNumber': '358', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-11T07:45:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T08:51:00'}, 'flightNumber': '4645', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '666.77', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T12:35:00'}, 'arrival': {'iataCode': 'IAD', 'at': '2024-03-10T16:20:00'}, 'flightNumber': '358', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-11T10:45:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T11:56:00'}, 'flightNumber': '4704', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '666.77', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T12:35:00'}, 'arrival': {'iataCode': 'IAD', 'at': '2024-03-10T16:20:00'}, 'flightNumber': '358', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-11T09:45:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T10:42:00'}, 'flightNumber': '5215', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '764.60', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T17:15:00'}, 'arrival': {'iataCode': 'IAD', 'at': '2024-03-10T21:10:00'}, 'flightNumber': '1240', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-11T09:45:00'}, 'arrival': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T10:42:00'}, 'flightNumber': '5215', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'ORD', 'terminal': '2', 'at': '2024-03-11T12:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T14:19:00'}, 'flightNumber': '5728', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '765.35', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T16:42:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T17:49:00'}, 'flightNumber': '2655', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-11T15:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T18:12:00'}, 'flightNumber': '4252', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '765.35', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T16:42:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T17:49:00'}, 'flightNumber': '2655', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-11T17:45:00'}, 'arrival': {'iataCode': 'LNK', 'at': '2024-03-11T20:07:00'}, 'flightNumber': '4910', 'carrier': 'UNITED AIRLINES'}]}][0m[32;1m[1;3m最早的航班是2024年3月10日，从德克萨斯州达拉斯出发，前往内布拉斯加州林肯，预计在2024年3月11日14:19抵达内布拉斯加州。

最终答案：最早的航班是2024年3月10日，从德克萨斯州达拉斯出发，前往内布拉斯加州林肯，预计在2024年3月11日14:19抵达内布拉斯加州。[0m

[1m> 完成链。[0m
```


```output
{'input': '2024年3月10日，从德克萨斯州达拉斯出发，前往内布拉斯加州林肯的最早航班在内布拉斯加州的降落时间是什么？',
 'output': '最早的航班是2024年3月10日，从德克萨斯州达拉斯出发，前往内布拉斯加州林肯，预计在2024年3月11日14:19抵达内布拉斯加州。'}
```



```python
# 为了正确执行API，请将查询日期更改为特征
agent_executor.invoke(
    {
        "input": "2024年3月10日，从俄勒冈州波特兰到德克萨斯州达拉斯的最便宜航班的全程时间是多少？"
    }
)
```
```output


[1m> 进入新的AgentExecutor链...[0m
[32;1m[1;3m问题：2024年3月10日，从俄勒冈州波特兰到德克萨斯州达拉斯的最便宜航班的全程时间是多少？
思考：我们需要找到2024年3月10日从俄勒冈州波特兰到德克萨斯州达拉斯的最便宜航班，然后计算总旅行时间。
行动：
```
{
  "action": "single_flight_search",
  "action_input": {
    "originLocationCode": "PDX",
    "destinationLocationCode": "DFW",
    "departureDateTimeEarliest": "2024-03-10T00:00:00",
    "departureDateTimeLatest": "2024-03-10T23:59:59"
  }
}
```

"action": "single_flight_search",
  "action_input": {
    "originLocationCode": "DFW",
    "destinationLocationCode": "DCA",
    "departureDateTimeEarliest": "2024-03-10T00:00:00",
    "departureDateTimeLatest": "2024-03-10T23:59:59"
  }
}
``` 
```output
[{'price': {'total': '303.31', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:00:00'}, 'arrival': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T10:25:00'}, 'flightNumber': '1517', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T12:00:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-10T13:19:00'}, 'flightNumber': '4431', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '405.43', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:00:00'}, 'arrival': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T10:25:00'}, 'flightNumber': '1517', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T14:00:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-10T15:18:00'}, 'flightNumber': '4433', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '539.81', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T08:59:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T10:22:00'}, 'flightNumber': '2147', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T12:32:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-10T17:53:00'}, 'flightNumber': '688', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T11:00:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T12:17:00'}, 'flightNumber': '2549', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T16:40:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-10T21:59:00'}, 'flightNumber': '690', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:15:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T07:27:00'}, 'flightNumber': '1323', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T12:32:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-10T17:53:00'}, 'flightNumber': '688', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T08:59:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T10:22:00'}, 'flightNumber': '2147', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T16:40:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-10T21:59:00'}, 'flightNumber': '690', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T23:29:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-11T00:43:00'}, 'flightNumber': '4463', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-11T07:30:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-11T12:56:00'}, 'flightNumber': '686', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:15:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T07:27:00'}, 'flightNumber': '1323', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T16:40:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-10T21:59:00'}, 'flightNumber': '690', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T20:56:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T22:12:00'}, 'flightNumber': '4069', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-11T07:30:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-11T12:56:00'}, 'flightNumber': '686', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T23:29:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-11T00:43:00'}, 'flightNumber': '4463', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-11T12:32:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-11T17:53:00'}, 'flightNumber': '688', 'carrier': 'FRONTIER AIRLINES'}]}]
```
```output
我们已经找到2024年3月10日从DFW到DCA的多个航班选项。我们需要选择最早的航班，并在邮件中包含所有必要的航班细节给保罗。

行动：
```

"action": "single_flight_search",
  "action_input": {
    "originLocationCode": "DFW",
    "destinationLocationCode": "DCA",
    "departureDateTimeEarliest": "2024-03-10T00:00:00",
    "departureDateTimeLatest": "2024-03-10T23:59:59"
  }
}
```
[0m[33;1m[1;3m[{'price': {'total': '303.31', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:00:00'}, 'arrival': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T10:25:00'}, 'flightNumber': '1517', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T12:00:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-10T13:19:00'}, 'flightNumber': '4431', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '405.43', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:00:00'}, 'arrival': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T10:25:00'}, 'flightNumber': '1517', 'carrier': 'UNITED AIRLINES'}, {'departure': {'iataCode': 'EWR', 'terminal': 'C', 'at': '2024-03-10T14:00:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '2', 'at': '2024-03-10T15:18:00'}, 'flightNumber': '4433', 'carrier': 'UNITED AIRLINES'}]}, {'price': {'total': '539.81', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T08:59:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T10:22:00'}, 'flightNumber': '2147', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T12:32:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-10T17:53:00'}, 'flightNumber': '688', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T11:00:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T12:17:00'}, 'flightNumber': '2549', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T16:40:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-10T21:59:00'}, 'flightNumber': '690', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:15:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T07:27:00'}, 'flightNumber': '1323', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T12:32:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-10T17:53:00'}, 'flightNumber': '688', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T08:59:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T10:22:00'}, 'flightNumber': '2147', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T16:40:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-10T21:59:00'}, 'flightNumber': '690', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T23:29:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-11T00:43:00'}, 'flightNumber': '4463', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-11T07:30:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-11T12:56:00'}, 'flightNumber': '686', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T06:15:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T07:27:00'}, 'flightNumber': '1323', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-10T16:40:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-10T21:59:00'}, 'flightNumber': '690', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T20:56:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-10T22:12:00'}, 'flightNumber': '4069', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-11T07:30:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-11T12:56:00'}, 'flightNumber': '686', 'carrier': 'FRONTIER AIRLINES'}]}, {'price': {'total': '544.98', 'currency': 'EURO'}, 'segments': [{'departure': {'iataCode': 'DFW', 'terminal': 'E', 'at': '2024-03-10T23:29:00'}, 'arrival': {'iataCode': 'DEN', 'at': '2024-03-11T00:43:00'}, 'flightNumber': '4463', 'carrier': 'FRONTIER AIRLINES'}, {'departure': {'iataCode': 'DEN', 'at': '2024-03-11T12:32:00'}, 'arrival': {'iataCode': 'DCA', 'terminal': '1', 'at': '2024-03-11T17:53:00'}, 'flightNumber': '688', 'carrier': 'FRONTIER AIRLINES'}]}][0m[32;1m[1;3m我们已找到2024年3月10日从达拉斯沃斯堡（DFW）到华盛顿特区（DCA）的多个航班选项。最早的航班是联合航空的航班，计划于06:00从DFW起飞，并于13:19到达DCA，航班号为1517和4431。总价为303.31欧元。[0m

[1m> Finished chain.[0m
```

```output
{'input': "请草拟一封简洁的电子邮件，从圣地亚哥发给保罗，圣地亚哥的旅行代理人，请求他预订2024年3月10日从DFW到DCA的最早航班。邮件中包含所有航班详细信息。",
 'output': '我们找到了一些2024年3月10日从达拉斯沃斯堡（DFW）到华盛顿特区（DCA）的航班选项。最早的航班是美联航，06:00从DFW起飞，13:19到达DCA，航班号为1517和4431。总价格为303.31欧元。'}
```