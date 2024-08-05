---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/nasa.ipynb
---

# NASA

本笔记本展示了如何使用代理与NASA工具包进行交互。该工具包提供了对NASA图像和视频库API的访问，未来版本有可能扩展并包括其他可访问的NASA API。

**注意：当未指定所需媒体结果的数量时，NASA图像和视频库的搜索查询可能会产生大量响应。在使用具有LLM令牌额度的代理之前，请考虑这一点。**

## 示例用法:
---

### 初始化代理


```python
%pip install -qU langchain-community
```


```python
from langchain.agents import AgentType, initialize_agent
from langchain_community.agent_toolkits.nasa.toolkit import NasaToolkit
from langchain_community.utilities.nasa import NasaAPIWrapper
from langchain_openai import OpenAI

llm = OpenAI(temperature=0, openai_api_key="")
nasa = NasaAPIWrapper()
toolkit = NasaToolkit.from_nasa_api_wrapper(nasa)
agent = initialize_agent(
    toolkit.get_tools(), llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)
```

### 查询媒体资产


```python
agent.run(
    "Can you find three pictures of the moon published between the years 2014 and 2020?"
)
```

### 查询媒体资产的详细信息


```python
output = agent.run(
    "I've just queried an image of the moon with the NASA id NHQ_2019_0311_Go Forward to the Moon."
    " Where can I find the metadata manifest for this asset?"
)
```