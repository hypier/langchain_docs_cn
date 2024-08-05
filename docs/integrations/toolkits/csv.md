---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/csv.ipynb
---

# CSV

本笔记本展示了如何使用代理与 `CSV` 格式的数据进行交互。它主要优化用于问答。

**注意：此代理在后台调用 Pandas DataFrame 代理，后者又调用 Python 代理，执行 LLM 生成的 Python 代码 - 如果 LLM 生成的 Python 代码有害，这可能会很糟糕。请谨慎使用。**




```python
from langchain.agents.agent_types import AgentType
from langchain_experimental.agents.agent_toolkits import create_csv_agent
from langchain_openai import ChatOpenAI, OpenAI
```

## 使用 `ZERO_SHOT_REACT_DESCRIPTION`

这展示了如何使用 `ZERO_SHOT_REACT_DESCRIPTION` 代理类型初始化代理。

```python
agent = create_csv_agent(
    OpenAI(temperature=0),
    "titanic.csv",
    verbose=True,
    agent_type=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
)
```

## 使用 OpenAI 函数

这显示了如何使用 OPENAI_FUNCTIONS 代理类型初始化代理。请注意，这是上述内容的替代方案。

```python
agent = create_csv_agent(
    ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0613"),
    "titanic.csv",
    verbose=True,
    agent_type=AgentType.OPENAI_FUNCTIONS,
)
```

```python
agent.run("how many rows are there?")
```
```output
Error in on_chain_start callback: 'name'
``````output
[32;1m[1;3m
Invoking: `python_repl_ast` with `df.shape[0]`


[0m[36;1m[1;3m891[0m[32;1m[1;3m数据框中有 891 行。[0m

[1m> Finished chain.[0m
```


```output
'数据框中有 891 行。'
```



```python
agent.run("how many people have more than 3 siblings")
```
```output
Error in on_chain_start callback: 'name'
``````output
[32;1m[1;3m
Invoking: `python_repl_ast` with `df[df['SibSp'] > 3]['PassengerId'].count()`


[0m[36;1m[1;3m30[0m[32;1m[1;3m数据框中有 30 人有超过 3 个兄弟姐妹。[0m

[1m> Finished chain.[0m
```


```output
'数据框中有 30 人有超过 3 个兄弟姐妹。'
```



```python
agent.run("whats the square root of the average age?")
```
```output
Error in on_chain_start callback: 'name'
``````output
[32;1m[1;3m
Invoking: `python_repl_ast` with `import pandas as pd
import math

# Create a dataframe
data = {'Age': [22, 38, 26, 35, 35]}
df = pd.DataFrame(data)

# Calculate the average age
average_age = df['Age'].mean()

# Calculate the square root of the average age
square_root = math.sqrt(average_age)

square_root`


[0m[36;1m[1;3m5.585696017507576[0m[32;1m[1;3m平均年龄的平方根大约是 5.59。[0m

[1m> Finished chain.[0m
```


```output
'平均年龄的平方根大约是 5.59。'
```

### 多个 CSV 示例

下一部分展示了代理如何与作为列表传入的多个 csv 文件进行交互。

```python
agent = create_csv_agent(
    ChatOpenAI(temperature=0, model="gpt-3.5-turbo-0613"),
    ["titanic.csv", "titanic_age_fillna.csv"],
    verbose=True,
    agent_type=AgentType.OPENAI_FUNCTIONS,
)
agent.run("how many rows in the age column are different between the two dfs?")
```
```output
Error in on_chain_start callback: 'name'
``````output
[32;1m[1;3m
Invoking: `python_repl_ast` with `df1['Age'].nunique() - df2['Age'].nunique()`


[0m[36;1m[1;3m-1[0m[32;1m[1;3m在两个数据框的年龄列中有 1 行不同。[0m

[1m> Finished chain.[0m
```


```output
'在两个数据框的年龄列中有 1 行不同。'
```