---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/pandas.ipynb
---

# Pandas Dataframe

本笔记本展示了如何使用代理与 `Pandas DataFrame` 进行交互。它主要针对问答进行优化。

**注意：此代理在后台调用 `Python` 代理，执行 LLM 生成的 Python 代码 - 如果 LLM 生成的 Python 代码有害，这可能是危险的。请谨慎使用。**


```python
from langchain.agents.agent_types import AgentType
from langchain_experimental.agents.agent_toolkits import create_pandas_dataframe_agent
from langchain_openai import ChatOpenAI
```


```python
import pandas as pd
from langchain_openai import OpenAI

df = pd.read_csv(
    "https://raw.githubusercontent.com/pandas-dev/pandas/main/doc/data/titanic.csv"
)
```

## 使用 `ZERO_SHOT_REACT_DESCRIPTION`

这展示了如何使用 `ZERO_SHOT_REACT_DESCRIPTION` 代理类型初始化代理。

```python
agent = create_pandas_dataframe_agent(OpenAI(temperature=0), df, verbose=True)
```

## 使用 OpenAI 函数

这展示了如何使用 OPENAI_FUNCTIONS 代理类型初始化代理。请注意，这是一种替代上述方法。


```python
agent = create_pandas_dataframe_agent(
    ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0613"),
    df,
    verbose=True,
    agent_type=AgentType.OPENAI_FUNCTIONS,
)
```


```python
agent.invoke("有多少行？")
```
```output


[1m> Entering new  chain...[0m
[32;1m[1;3m
Invoking: `python_repl_ast` with `df.shape[0]`


[0m[36;1m[1;3m891[0m[32;1m[1;3m数据框中有 891 行。[0m

[1m> Finished chain.[0m
```


```output
'数据框中有 891 行。'
```



```python
agent.invoke("有多少人有超过 3 个兄弟姐妹")
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3mThought: 我需要计算有超过 3 个兄弟姐妹的人数
Action: python_repl_ast
Action Input: df[df['SibSp'] > 3].shape[0][0m
Observation: [36;1m[1;3m30[0m
Thought:[32;1m[1;3m 我现在知道最终答案
Final Answer: 30 人有超过 3 个兄弟姐妹。[0m

[1m> Finished chain.[0m
```


```output
'30 人有超过 3 个兄弟姐妹。'
```



```python
agent.invoke("平均年龄的平方根是多少？")
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3mThought: 我需要先计算平均年龄
Action: python_repl_ast
Action Input: df['Age'].mean()[0m
Observation: [36;1m[1;3m29.69911764705882[0m
Thought:[32;1m[1;3m 我现在需要计算平均年龄的平方根
Action: python_repl_ast
Action Input: math.sqrt(df['Age'].mean())[0m
Observation: [36;1m[1;3mNameError("name 'math' is not defined")[0m
Thought:[32;1m[1;3m 我需要导入 math 库
Action: python_repl_ast
Action Input: import math[0m
Observation: [36;1m[1;3m[0m
Thought:[32;1m[1;3m 我现在需要计算平均年龄的平方根
Action: python_repl_ast
Action Input: math.sqrt(df['Age'].mean())[0m
Observation: [36;1m[1;3m5.449689683556195[0m
Thought:[32;1m[1;3m 我现在知道最终答案
Final Answer: 平均年龄的平方根是 5.449689683556195。[0m

[1m> Finished chain.[0m
```


```output
'平均年龄的平方根是 5.449689683556195。'
```

## 多数据框示例

下一部分展示了代理如何与作为列表传入的多个数据框进行交互。

```python
df1 = df.copy()
df1["Age"] = df1["Age"].fillna(df1["Age"].mean())
```

```python
agent = create_pandas_dataframe_agent(OpenAI(temperature=0), [df, df1], verbose=True)
agent.invoke("how many rows in the age column are different?")
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3mThought: I need to compare the age columns in both dataframes
Action: python_repl_ast
Action Input: len(df1[df1['Age'] != df2['Age']])[0m
Observation: [36;1m[1;3m177[0m
Thought:[32;1m[1;3m I now know the final answer
Final Answer: 177 rows in the age column are different.[0m

[1m> Finished chain.[0m
```

```output
'177 rows in the age column are different.'
```