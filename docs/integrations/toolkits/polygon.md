---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/polygon.ipynb
---
# Polygon IO Toolkit

This notebook shows how to use agents to interact with the [Polygon IO](https://polygon.io/) toolkit. The toolkit provides access to Polygon's Stock Market Data API.

## Example Use


### Setup


```python
%pip install --upgrade --quiet langchain-community > /dev/null
```

Get your Polygon IO API key [here](https://polygon.io/), and then set it below.
Note that the tool used in this example requires a "Stocks Advanced" subscription


```python
import getpass
import os

os.environ["POLYGON_API_KEY"] = getpass.getpass()
```
```output
········
```
It's also helpful (but not needed) to set up [LangSmith](https://smith.langchain.com/) for best-in-class observability


```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

### Initializing the agent


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

### Get the last price quote for a stock


```python
agent_executor.invoke({"input": "What is the latest stock price for AAPL?"})
```
