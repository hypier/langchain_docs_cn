---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/writer.ipynb
---

# Writer

[Writer](https://writer.com/) 是一个生成不同语言内容的平台。

本示例介绍如何使用 LangChain 与 `Writer` [模型](https://dev.writer.com/docs/models) 进行交互。

您可以在此处获取 WRITER_API_KEY [here](https://dev.writer.com/docs)。

```python
from getpass import getpass

WRITER_API_KEY = getpass()
```
```output
 ········
```

```python
import os

os.environ["WRITER_API_KEY"] = WRITER_API_KEY
```


```python
from langchain.chains import LLMChain
from langchain_community.llms import Writer
from langchain_core.prompts import PromptTemplate
```


```python
template = """Question: {question}

Answer: Let's think step by step."""

prompt = PromptTemplate.from_template(template)
```


```python
# 如果您遇到错误，可能需要设置 "base_url" 参数，该参数可以从错误日志中获取。

llm = Writer()
```


```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```


```python
question = "What NFL team won the Super Bowl in the year Justin Beiber was born?"

llm_chain.run(question)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)