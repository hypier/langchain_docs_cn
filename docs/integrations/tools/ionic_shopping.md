---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/ionic_shopping.ipynb
---

# Ionic购物工具

[Ionic](https://www.ioniccommerce.com/)是一个即插即用的电子商务市场，专为AI助手设计。通过在您的代理中包含[Ionic Tool](https://github.com/ioniccommerce/ionic_langchain)，您可以轻松地为用户提供直接在您的代理中购物和交易的能力，并且您将获得交易的一部分。

这是一个基本的Jupyter Notebook，演示如何将Ionic Tool集成到您的代理中。有关如何使用Ionic设置代理的更多信息，请参阅Ionic [文档](https://docs.ioniccommerce.com/introduction)。

此Jupyter Notebook演示如何将Ionic工具与代理一起使用。

**注意：ionic-langchain包由Ionic Commerce团队维护，而非LangChain维护者。**



---

## 设置


```python
pip install langchain langchain_openai langchainhub
```


```python
pip install ionic-langchain
```

## 设置代理


```python
from ionic_langchain.tool import Ionic, IonicTool
from langchain import hub
from langchain.agents import AgentExecutor, Tool, create_react_agent
from langchain_openai import OpenAI

# 基于 ReAct 代理
# https://python.langchain.com/docs/modules/agents/agent_types/react
# 请参见论文 "ReAct: Synergizing Reasoning and Acting in Language Models" (https://arxiv.org/abs/2210.03629)
# 如需帮助，请联系 support@ionicapi.com 获取其他代理类型的支持。

open_ai_key = "YOUR KEY HERE"
model = "gpt-3.5-turbo-instruct"
temperature = 0.6

llm = OpenAI(openai_api_key=open_ai_key, model_name=model, temperature=temperature)


ionic_tool = IonicTool().tool()


# 该工具自带提示，
# 但您也可以通过 description 属性直接更新它：

ionic_tool.description = str(
    """
Ionic 是一个电子商务购物工具。助手使用 Ionic 电子商务购物工具来查找、发现和比较来自数千家在线零售商的产品。当用户寻找产品推荐或试图找到特定产品时，助手应使用该工具。

用户可以指定他们希望看到的结果数量、最低价格和最高价格。
Ionic 工具输入是一个以逗号分隔的值字符串：
  - 查询字符串（必需，不得包含逗号）
  - 结果数量（默认为 4，不超过 10）
  - 最低价格（以美分为单位，$5 为 500）
  - 最高价格（以美分为单位）
例如，如果寻找价格在 5 到 10 美元之间的咖啡豆，工具输入将是 `coffee beans, 5, 500, 1000`。

将它们作为 markdown 格式的列表返回，每个推荐来自工具结果，确保包含完整的 PDP URL。例如：

1. 产品 1: [价格] -- 链接
2. 产品 2: [价格] -- 链接
3. 产品 3: [价格] -- 链接
4. 产品 4: [价格] -- 链接
"""
)

tools = [ionic_tool]

# create_react_agent 的默认提示
prompt = hub.pull("hwchase17/react")

agent = create_react_agent(
    llm,
    tools,
    prompt=prompt,
)

agent_executor = AgentExecutor(
    agent=agent, tools=tools, handle_parsing_errors=True, verbose=True, max_iterations=5
)
```

## 运行


```python
input = (
    "I'm looking for a new 4k monitor can you find me some options for less than $1000"
)
agent_executor.invoke({"input": input})
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)