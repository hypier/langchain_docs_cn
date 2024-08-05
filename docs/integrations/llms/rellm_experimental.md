---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/rellm_experimental.ipynb
---

# RELLM

[RELLM](https://github.com/r2d4/rellm) 是一个将本地 Hugging Face pipeline 模型封装用于结构化解码的库。

它通过一次生成一个 token 的方式工作。在每一步中，它会屏蔽不符合提供的部分正则表达式的 tokens。

**警告 - 该模块仍处于实验阶段**

```python
%pip install --upgrade --quiet  rellm langchain-huggingface > /dev/null
```

### Hugging Face Baseline

首先，让我们通过检查模型的输出，建立一个定性基准，而不使用结构化解码。


```python
import logging

logging.basicConfig(level=logging.ERROR)
prompt = """Human: "What's the capital of the United States?"
AI Assistant:{
  "action": "Final Answer",
  "action_input": "The capital of the United States is Washington D.C."
}
Human: "What's the capital of Pennsylvania?"
AI Assistant:{
  "action": "Final Answer",
  "action_input": "The capital of Pennsylvania is Harrisburg."
}
Human: "What 2 + 5?"
AI Assistant:{
  "action": "Final Answer",
  "action_input": "2 + 5 = 7."
}
Human: 'What's the capital of Maryland?'
AI Assistant:"""
```


```python
from langchain_huggingface import HuggingFacePipeline
from transformers import pipeline

hf_model = pipeline(
    "text-generation", model="cerebras/Cerebras-GPT-590M", max_new_tokens=200
)

original_model = HuggingFacePipeline(pipeline=hf_model)

generated = original_model.generate([prompt], stop=["Human:"])
print(generated)
```
```output
Setting `pad_token_id` to `eos_token_id`:50256 for open-end generation.
``````output
generations=[[Generation(text=' "What\'s the capital of Maryland?"\n', generation_info=None)]] llm_output=None
```
***这可真不怎么样，是吧？它没有回答问题，而且根本没有遵循 JSON 格式！让我们尝试使用结构化解码。***

## RELLM LLM Wrapper

让我们再试一次，这次提供一个正则表达式来匹配JSON结构格式。

```python
import regex  # Note this is the regex library NOT python's re stdlib module

# We'll choose a regex that matches to a structured json string that looks like:
# {
#  "action": "Final Answer",
# "action_input": string or dict
# }
pattern = regex.compile(
    r'\{\s*"action":\s*"Final Answer",\s*"action_input":\s*(\{.*\}|"[^"]*")\s*\}\nHuman:'
)
```

```python
from langchain_experimental.llms import RELLM

model = RELLM(pipeline=hf_model, regex=pattern, max_new_tokens=200)

generated = model.predict(prompt, stop=["Human:"])
print(generated)
```
```output
{"action": "Final Answer",
  "action_input": "The capital of Maryland is Baltimore."
}
```
**太好了！没有解析错误。**

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)