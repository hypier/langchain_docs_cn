---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/powerbi.ipynb
---

# PowerBI 数据集

本笔记本展示了一个代理与 `Power BI 数据集` 交互的过程。代理正在回答有关数据集的更一般性问题，并从错误中恢复。

请注意，由于该代理正在积极开发中，所有答案可能并不正确。它运行在 [executequery 端点](https://learn.microsoft.com/en-us/rest/api/power-bi/datasets/execute-queries) 上，该端点不允许删除。

### 注意事项：
- 它依赖于使用 azure.identity 包进行身份验证，可以通过 `pip install azure-identity` 安装。或者，您可以使用字符串形式的令牌创建 Power BI 数据集，而无需提供凭据。
- 您还可以提供一个用户名，以便在启用 RLS 的数据集上进行模拟使用。
- 工具包使用 LLM 从问题中创建查询，代理使用 LLM 进行整体执行。
- 测试主要是在 `gpt-3.5-turbo-instruct` 模型上进行的，codex 模型的表现似乎不太理想。

## 初始化


```python
from azure.identity import DefaultAzureCredential
from langchain_community.agent_toolkits import PowerBIToolkit, create_pbi_agent
from langchain_community.utilities.powerbi import PowerBIDataset
from langchain_openai import ChatOpenAI
```


```python
fast_llm = ChatOpenAI(
    temperature=0.5, max_tokens=1000, model_name="gpt-3.5-turbo", verbose=True
)
smart_llm = ChatOpenAI(temperature=0, max_tokens=100, model_name="gpt-4", verbose=True)

toolkit = PowerBIToolkit(
    powerbi=PowerBIDataset(
        dataset_id="<dataset_id>",
        table_names=["table1", "table2"],
        credential=DefaultAzureCredential(),
    ),
    llm=smart_llm,
)

agent_executor = create_pbi_agent(
    llm=fast_llm,
    toolkit=toolkit,
    verbose=True,
)
```

## 示例：描述一个表格


```python
agent_executor.run("Describe table1")
```

## 示例：对表的简单查询
在这个例子中，代理实际上找出了正确的查询，以获取表的行数。

```python
agent_executor.run("How many records are in table1?")
```

## 示例：运行查询


```python
agent_executor.run("How many records are there by dimension1 in table2?")
```


```python
agent_executor.run("What unique values are there for dimensions2 in table2")
```

## 示例：添加您自己的少量提示


```python
# 虚构示例
few_shots = """
问题：表 revenue 中有多少行？
DAX：EVALUATE ROW("行数", COUNTROWS(revenue_details))
----
问题：表 revenue 中年份不为空的行数是多少？
DAX：EVALUATE ROW("行数", COUNTROWS(FILTER(revenue_details, revenue_details[year] <> "")))
----
问题：revenue 中的平均值是多少（以美元计）？
DAX：EVALUATE ROW("平均值", AVERAGE(revenue_details[dollar_value]))
----
"""
toolkit = PowerBIToolkit(
    powerbi=PowerBIDataset(
        dataset_id="<dataset_id>",
        table_names=["table1", "table2"],
        credential=DefaultAzureCredential(),
    ),
    llm=smart_llm,
    examples=few_shots,
)
agent_executor = create_pbi_agent(
    llm=fast_llm,
    toolkit=toolkit,
    verbose=True,
)
```


```python
agent_executor.run("2022年 revenue 中的最大值是多少（以美元计）？")
```