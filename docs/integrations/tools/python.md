---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/python.ipynb
---

# Python REPL

有时，对于复杂的计算，与其让 LLM 直接生成答案，不如让 LLM 生成代码来计算答案，然后运行该代码以获取答案。为了方便实现这一点，我们提供了一个简单的 Python REPL 来执行命令。

该接口只会返回打印的内容——因此，如果您想用它来计算答案，请确保打印出答案。


:::caution
Python REPL 可以在主机上执行任意代码（例如，删除文件，进行网络请求）。请谨慎使用。

有关一般安全指南的更多信息，请参见 https://python.langchain.com/v0.2/docs/security/.
:::


```python
from langchain_core.tools import Tool
from langchain_experimental.utilities import PythonREPL
```


```python
python_repl = PythonREPL()
```


```python
python_repl.run("print(1+1)")
```
```output
Python REPL can execute arbitrary code. Use with caution.
```


```output
'2\n'
```



```python
# You can create the tool to pass to an agent
repl_tool = Tool(
    name="python_repl",
    description="一个 Python shell。使用它来执行 Python 命令。输入应该是有效的 Python 命令。如果您想查看值的输出，应该使用 `print(...)` 将其打印出来。",
    func=python_repl.run,
)
```

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)