---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/solar.ipynb
---
# Solar

*This community integration is deprecated. You should use [`ChatUpstage`](../../chat/upstage) instead to access Solar LLM via the chat model connector.*


```python
import os

from langchain_community.llms.solar import Solar

os.environ["SOLAR_API_KEY"] = "SOLAR_API_KEY"
llm = Solar()
llm.invoke("tell me a story?")
```


```python
from langchain.chains import LLMChain
from langchain_community.llms.solar import Solar
from langchain_core.prompts import PromptTemplate

template = """Question: {question}

Answer: Let's think step by step."""

prompt = PromptTemplate.from_template(template)

llm = Solar()
llm_chain = LLMChain(prompt=prompt, llm=llm)

question = "What NFL team won the Super Bowl in the year Justin Beiber was born?"

llm_chain.run(question)
```


## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)