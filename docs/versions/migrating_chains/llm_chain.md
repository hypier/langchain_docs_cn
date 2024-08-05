---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/versions/migrating_chains/llm_chain.ipynb
title: 从 LLMChain 迁移
---

[`LLMChain`](https://api.python.langchain.com/en/latest/chains/langchain.chains.llm.LLMChain.html) 将提示模板、LLM 和输出解析器结合成一个类。

切换到 LCEL 实现的一些优势包括：

- 对内容和参数的清晰理解。遗留的 `LLMChain` 包含默认的输出解析器和其他选项。
- 更容易的流式处理。`LLMChain` 仅通过回调支持流式处理。
- 更容易访问原始消息输出（如果需要）。`LLMChain` 仅通过参数或回调暴露这些内容。


```python
%pip install --upgrade --quiet langchain-openai
```


```python
import os
from getpass import getpass

os.environ["OPENAI_API_KEY"] = getpass()
```

import { ColumnContainer, Column } from "@theme/Columns";

<ColumnContainer>

<Column>

#### 遗留



```python
from langchain.chains import LLMChain
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages(
    [("user", "Tell me a {adjective} joke")],
)

chain = LLMChain(llm=ChatOpenAI(), prompt=prompt)

chain({"adjective": "funny"})
```



```output
{'adjective': 'funny',
 'text': "Why couldn't the bicycle stand up by itself?\n\nBecause it was two tired!"}
```



</Column>

<Column>

#### LCEL




```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages(
    [("user", "Tell me a {adjective} joke")],
)

chain = prompt | ChatOpenAI() | StrOutputParser()

chain.invoke({"adjective": "funny"})
```



```output
'Why was the math book sad?\n\nBecause it had too many problems.'
```



</Column>
</ColumnContainer>

请注意，`LLMChain` 默认返回一个包含输入和输出的 `dict`。如果需要这种行为，我们可以使用另一个 LCEL 原语 [`RunnablePassthrough`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.passthrough.RunnablePassthrough.html) 来复制它：


```python
from langchain_core.runnables import RunnablePassthrough

outer_chain = RunnablePassthrough().assign(text=chain)

outer_chain.invoke({"adjective": "funny"})
```



```output
{'adjective': 'funny',
 'text': 'Why did the scarecrow win an award? Because he was outstanding in his field!'}
```

## 下一步

查看 [本教程](/docs/tutorials/llm_chain) 以获取有关使用提示模板、LLMs 和输出解析器的更多详细信息。

查看 [LCEL 概念文档](/docs/concepts/#langchain-expression-language-lcel) 以获取更多背景信息。