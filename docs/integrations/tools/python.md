---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/python.ipynb
---
# Python REPL

Sometimes, for complex calculations, rather than have an LLM generate the answer directly, it can be better to have the LLM generate code to calculate the answer, and then run that code to get the answer. In order to easily do that, we provide a simple Python REPL to execute commands in.

This interface will only return things that are printed - therefore, if you want to use it to calculate an answer, make sure to have it print out the answer.


:::caution
Python REPL can execute arbitrary code on the host machine (e.g., delete files, make network requests). Use with caution.

For more information general security guidelines, please see https://python.langchain.com/v0.2/docs/security/.
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
    description="A Python shell. Use this to execute python commands. Input should be a valid python command. If you want to see the output of a value, you should print it out with `print(...)`.",
    func=python_repl.run,
)
```


## Related

- Tool [conceptual guide](/docs/concepts/#tools)
- Tool [how-to guides](/docs/how_to/#tools)