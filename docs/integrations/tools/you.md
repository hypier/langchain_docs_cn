---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/you.ipynb
---

# You.com 搜索

[you.com API](https://api.you.com) 是一套工具，旨在帮助开发人员将大型语言模型（LLMs）的输出与最新、最准确、最相关的信息相结合，这些信息可能未包含在其训练数据集中。

## 设置

该工具位于 `langchain-community` 包中。

您还需要设置您的 you.com API 密钥。


```python
%pip install --upgrade --quiet langchain-community
```


```python
import os

os.environ["YDC_API_KEY"] = ""

# 用于链式调用部分
os.environ["OPENAI_API_KEY"] = ""

## 替代方案：从 .env 文件加载 YDC_API_KEY

# !pip install --quiet -U python-dotenv
# import dotenv
# dotenv.load_dotenv()
```

设置 [LangSmith](https://smith.langchain.com/) 以获得最佳的可观察性也是有帮助的（但不是必需的）


```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 工具使用


```python
from langchain_community.tools.you import YouSearchTool
from langchain_community.utilities.you import YouSearchAPIWrapper

api_wrapper = YouSearchAPIWrapper(num_web_results=1)
tool = YouSearchTool(api_wrapper=api_wrapper)

tool
```



```output
YouSearchTool(api_wrapper=YouSearchAPIWrapper(ydc_api_key='054da371-e73b-47c1-a6d9-3b0cddf0fa3e<__>1Obt7EETU8N2v5f4MxaH0Zhx', num_web_results=1, safesearch=None, country=None, k=None, n_snippets_per_hit=None, endpoint_type='search', n_hits=None))
```



```python
# .invoke wraps utility.results
response = tool.invoke("What is the weather in NY")

# .invoke should have a Document for each `snippet`
print(len(response))

for item in response:
    print(item)
```

## 链接

我们在这里展示如何将其作为 [agent](/docs/tutorials/agents) 的一部分使用。我们使用 OpenAI Functions Agent，因此我们需要设置和安装所需的依赖项。我们还将使用 [LangSmith Hub](https://smith.langchain.com/hub) 来获取提示，因此我们需要安装它。


```python
# you need a model to use in the chain
!pip install --upgrade --quiet langchain langchain-openai langchainhub langchain-community
```


```python
from langchain import hub
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI

instructions = """You are an assistant."""
base_prompt = hub.pull("langchain-ai/openai-functions-template")
prompt = base_prompt.partial(instructions=instructions)
llm = ChatOpenAI(temperature=0)
you_tool = YouSearchTool(api_wrapper=YouSearchAPIWrapper(num_web_results=1))
tools = [you_tool]
agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
)
```


```python
agent_executor.invoke({"input": "What is the weather in NY today?"})
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)