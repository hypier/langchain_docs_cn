---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/polygon.ipynb
---

# Polygon IO 工具包

本笔记本展示了如何使用代理与 [Polygon IO](https://polygon.io/) 工具包进行交互。该工具包提供对 Polygon 股票市场数据 API 的访问。

## 示例用法

### 设置


```python
%pip install --upgrade --quiet langchain-community > /dev/null
```

在此处获取您的 Polygon IO API 密钥 [here](https://polygon.io/)，然后在下面设置它。
请注意，此示例中使用的工具需要“股票高级”订阅


```python
import getpass
import os

os.environ["POLYGON_API_KEY"] = getpass.getpass()
```
```output
········
```
设置 [LangSmith](https://smith.langchain.com/) 以获得最佳的可观察性也是有帮助的（但不是必需的）


```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

### 初始化代理


```python
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_community.agent_toolkits.polygon.toolkit import PolygonToolkit
from langchain_community.utilities.polygon import PolygonAPIWrapper
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(temperature=0)

instructions = """You are an assistant."""
base_prompt = hub.pull("langchain-ai/openai-functions-template")
prompt = base_prompt.partial(instructions=instructions)
```


```python
polygon = PolygonAPIWrapper()
toolkit = PolygonToolkit.from_polygon_api_wrapper(polygon)
agent = create_openai_functions_agent(llm, toolkit.get_tools(), prompt)
```


```python
agent_executor = AgentExecutor(
    agent=agent,
    tools=toolkit.get_tools(),
    verbose=True,
)
```

### 获取股票的最新报价


```python
agent_executor.invoke({"input": "What is the latest stock price for AAPL?"})
```