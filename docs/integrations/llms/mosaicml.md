---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/mosaicml.ipynb
---

# MosaicML

[MosaicML](https://docs.mosaicml.com/en/latest/inference.html) 提供了一个托管推理服务。您可以使用多种开源模型，或者部署您自己的模型。

这个示例介绍了如何使用 LangChain 与 MosaicML 推理进行文本补全的交互。


```python
# sign up for an account: https://forms.mosaicml.com/demo?utm_source=langchain

from getpass import getpass

MOSAICML_API_TOKEN = getpass()
```


```python
import os

os.environ["MOSAICML_API_TOKEN"] = MOSAICML_API_TOKEN
```


```python
from langchain.chains import LLMChain
from langchain_community.llms import MosaicML
from langchain_core.prompts import PromptTemplate
```


```python
template = """Question: {question}"""

prompt = PromptTemplate.from_template(template)
```


```python
llm = MosaicML(inject_instruction_format=True, model_kwargs={"max_new_tokens": 128})
```


```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```


```python
question = "What is one good reason why you should train a large language model on domain specific data?"

llm_chain.run(question)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)