---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/versions/migrating_chains/llm_chain.ipynb
title: Migrating from LLMChain
---

[`LLMChain`](https://api.python.langchain.com/en/latest/chains/langchain.chains.llm.LLMChain.html) combined a prompt template, LLM, and output parser into a class.

Some advantages of switching to the LCEL implementation are:

- Clarity around contents and parameters. The legacy `LLMChain` contains a default output parser and other options.
- Easier streaming. `LLMChain` only supports streaming via callbacks.
- Easier access to raw message outputs if desired. `LLMChain` only exposes these via a parameter or via callback.


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

#### Legacy



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

Note that `LLMChain` by default returns a `dict` containing both the input and the output. If this behavior is desired, we can replicate it using another LCEL primitive, [`RunnablePassthrough`](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.passthrough.RunnablePassthrough.html):


```python
from langchain_core.runnables import RunnablePassthrough

outer_chain = RunnablePassthrough().assign(text=chain)

outer_chain.invoke({"adjective": "funny"})
```



```output
{'adjective': 'funny',
 'text': 'Why did the scarecrow win an award? Because he was outstanding in his field!'}
```


## Next steps

See [this tutorial](/docs/tutorials/llm_chain) for more detail on building with prompt templates, LLMs, and output parsers.

Check out the [LCEL conceptual docs](/docs/concepts/#langchain-expression-language-lcel) for more background information.
