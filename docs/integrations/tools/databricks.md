---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/databricks.ipynb
---

# Databricks Unity Catalog (UC)

本笔记本展示如何将 UC 函数用作 LangChain 工具。

请参阅 Databricks 文档（[AWS](https://docs.databricks.com/en/sql/language-manual/sql-ref-syntax-ddl-create-sql-function.html)|[Azure](https://learn.microsoft.com/en-us/azure/databricks/sql/language-manual/sql-ref-syntax-ddl-create-sql-function)|[GCP](https://docs.gcp.databricks.com/en/sql/language-manual/sql-ref-syntax-ddl-create-sql-function.html)）了解如何在 UC 中创建 SQL 或 Python 函数。请勿跳过函数和参数注释，这对于 LLM 正确调用函数至关重要。

在这个示例笔记本中，我们创建一个简单的 Python 函数来执行任意代码，并将其用作 LangChain 工具：

```sql
CREATE FUNCTION main.tools.python_exec (
  code STRING COMMENT 'Python code to execute. Remember to print the final result to stdout.'
)
RETURNS STRING
LANGUAGE PYTHON
COMMENT 'Executes Python code and returns its stdout.'
AS $$
  import sys
  from io import StringIO
  stdout = StringIO()
  sys.stdout = stdout
  exec(code)
  return stdout.getvalue()
$$
```

它在 Databricks SQL 仓库中的安全隔离环境中运行。


```python
%pip install --upgrade --quiet databricks-sdk langchain-community mlflow
```


```python
from langchain_community.chat_models.databricks import ChatDatabricks

llm = ChatDatabricks(endpoint="databricks-meta-llama-3-70b-instruct")
```


```python
from langchain_community.tools.databricks import UCFunctionToolkit

tools = (
    UCFunctionToolkit(
        # You can find the SQL warehouse ID in its UI after creation.
        warehouse_id="xxxx123456789"
    )
    .include(
        # Include functions as tools using their qualified names.
        # You can use "{catalog_name}.{schema_name}.*" to get all functions in a schema.
        "main.tools.python_exec",
    )
    .get_tools()
)
```


```python
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant. Make sure to use tool for information.",
        ),
        ("placeholder", "{chat_history}"),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}"),
    ]
)

agent = create_tool_calling_agent(llm, tools, prompt)
```


```python
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
agent_executor.invoke({"input": "36939 * 8922.4"})
```
```output


[1m> Entering new AgentExecutor chain...[0m
[32;1m[1;3m
Invoking: `main__tools__python_exec` with `{'code': 'print(36939 * 8922.4)'}`


[0m[36;1m[1;3m{"format": "SCALAR", "value": "329584533.59999996\n", "truncated": false}[0m[32;1m[1;3m乘法 36939 * 8922.4 的结果是 329,584,533.60。[0m

[1m> Finished chain.[0m
```


```output
{'input': '36939 * 8922.4',
 'output': '乘法 36939 * 8922.4 的结果是 329,584,533.60.'}
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)