---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/textgen.ipynb
---

# TextGen

[GitHub:oobabooga/text-generation-webui](https://github.com/oobabooga/text-generation-webui) 一个用于运行大型语言模型的 Gradio Web 用户界面，如 LLaMA、llama.cpp、GPT-J、Pythia、OPT 和 GALACTICA。

本示例介绍了如何使用 LangChain 通过 `text-generation-webui` API 集成与 LLM 模型进行交互。

请确保您已配置 `text-generation-webui` 并安装了 LLM。建议通过适合您操作系统的 [一键安装程序](https://github.com/oobabooga/text-generation-webui#one-click-installers) 进行安装。

一旦 `text-generation-webui` 安装并通过 Web 界面确认工作，请通过 Web 模型配置选项卡启用 `api` 选项，或通过将运行时参数 `--api` 添加到您的启动命令中来启用。

## 设置 model_url 并运行示例


```python
model_url = "http://localhost:5000"
```


```python
from langchain.chains import LLMChain
from langchain.globals import set_debug
from langchain_community.llms import TextGen
from langchain_core.prompts import PromptTemplate

set_debug(True)

template = """Question: {question}

Answer: Let's think step by step."""


prompt = PromptTemplate.from_template(template)
llm = TextGen(model_url=model_url)
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "What NFL team won the Super Bowl in the year Justin Bieber was born?"

llm_chain.run(question)
```

### 流媒体版本

您需要安装 websocket-client 才能使用此功能。
`pip install websocket-client`


```python
model_url = "ws://localhost:5005"
```


```python
from langchain.chains import LLMChain
from langchain.globals import set_debug
from langchain_community.llms import TextGen
from langchain_core.callbacks import StreamingStdOutCallbackHandler
from langchain_core.prompts import PromptTemplate

set_debug(True)

template = """Question: {question}

Answer: Let's think step by step."""


prompt = PromptTemplate.from_template(template)
llm = TextGen(
    model_url=model_url, streaming=True, callbacks=[StreamingStdOutCallbackHandler()]
)
llm_chain = LLMChain(prompt=prompt, llm=llm)
question = "What NFL team won the Super Bowl in the year Justin Bieber was born?"

llm_chain.run(question)
```


```python
llm = TextGen(model_url=model_url, streaming=True)
for chunk in llm.stream("Ask 'Hi, how are you?' like a pirate:'", stop=["'", "\n"]):
    print(chunk, end="", flush=True)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)