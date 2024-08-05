---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/semanticscholar.ipynb
---

# Semantic Scholar API Tool

本笔记本演示了如何使用语义学者工具与代理。

```python
# start by installing semanticscholar api
%pip install --upgrade --quiet  semanticscholar
```


```python
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
```


```python
instructions = """You are an expert researcher."""
base_prompt = hub.pull("langchain-ai/openai-functions-template")
prompt = base_prompt.partial(instructions=instructions)
```


```python
llm = ChatOpenAI(temperature=0)
```


```python
from langchain_community.tools.semanticscholar.tool import SemanticScholarQueryRun

tools = [SemanticScholarQueryRun()]
```


```python
agent = create_openai_functions_agent(llm, tools, prompt)
```


```python
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
)
```


```python
agent_executor.invoke(
    {
        "input": "What are some biases in the large language models? How have people tried to mitigate them? "
        "show me a list of papers and techniques. Based on your findings write new research questions "
        "to work on. Break down the task into subtasks for search. Use the search tool"
    }
)
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)