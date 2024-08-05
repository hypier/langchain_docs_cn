---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/memory/motorhead_memory.ipynb
---

# Motörhead

>[Motörhead](https://github.com/getmetal/motorhead) 是用 Rust 实现的内存服务器。它在后台自动处理增量摘要，并允许无状态应用程序。

## 设置

请参阅 [Motörhead](https://github.com/getmetal/motorhead) 上的说明以在本地运行服务器。


```python
from langchain.memory.motorhead_memory import MotorheadMemory
```

## 示例


```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
from langchain_openai import OpenAI

template = """You are a chatbot having a conversation with a human.

{chat_history}
Human: {human_input}
AI:"""

prompt = PromptTemplate(
    input_variables=["chat_history", "human_input"], template=template
)
memory = MotorheadMemory(
    session_id="testing-1", url="http://localhost:8080", memory_key="chat_history"
)

await memory.init()
# loads previous state from Motörhead 🤘

llm_chain = LLMChain(
    llm=OpenAI(),
    prompt=prompt,
    verbose=True,
    memory=memory,
)
```


```python
llm_chain.run("hi im bob")
```
```output


[1m> Entering new LLMChain chain...[0m
Prompt after formatting:
[32;1m[1;3mYou are a chatbot having a conversation with a human.


Human: hi im bob
AI:[0m

[1m> Finished chain.[0m
```


```output
' 你好，鲍勃，很高兴见到你！你今天过得怎么样？'
```



```python
llm_chain.run("whats my name?")
```
```output


[1m> Entering new LLMChain chain...[0m
Prompt after formatting:
[32;1m[1;3mYou are a chatbot having a conversation with a human.

Human: hi im bob
AI:  你好，鲍勃，很高兴见到你！你今天过得怎么样？
Human: whats my name?
AI:[0m

[1m> Finished chain.[0m
```


```output
' 你说你的名字是鲍勃，对吗？'
```



```python
llm_chain.run("whats for dinner?")
```
```output


[1m> Entering new LLMChain chain...[0m
Prompt after formatting:
[32;1m[1;3mYou are a chatbot having a conversation with a human.

Human: hi im bob
AI:  你好，鲍勃，很高兴见到你！你今天过得怎么样？
Human: whats my name?
AI:  你说你的名字是鲍勃，对吗？
Human: whats for dinner?
AI:[0m

[1m> Finished chain.[0m
```


```output
"  对不起，我不太确定你在问什么。你能否重新表述一下你的问题？"
```