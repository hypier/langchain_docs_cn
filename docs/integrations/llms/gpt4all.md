---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/gpt4all.ipynb
---

# GPT4All

[GitHub:nomic-ai/gpt4all](https://github.com/nomic-ai/gpt4all) 是一个开源聊天机器人生态系统，基于大量干净的助手数据进行训练，包括代码、故事和对话。

本示例介绍如何使用 LangChain 与 `GPT4All` 模型进行交互。


```python
%pip install --upgrade --quiet langchain-community gpt4all
```

### 导入 GPT4All


```python
from langchain_community.llms import GPT4All
from langchain_core.prompts import PromptTemplate
```

### 设置要传递给 LLM 的问题


```python
template = """Question: {question}

Answer: Let's think step by step."""

prompt = PromptTemplate.from_template(template)
```

### 指定模型

要在本地运行，请下载兼容的 ggml 格式模型。

[gpt4all 页面](https://gpt4all.io/index.html) 有一个有用的 `Model Explorer` 部分：

* 选择一个感兴趣的模型
* 使用 UI 下载并将 `.bin` 文件移动到 `local_path`（如下所示）

有关更多信息，请访问 https://github.com/nomic-ai/gpt4all。

---

此集成尚不支持通过 [`.stream()`](https://python.langchain.com/v0.2/docs/how_to/streaming/) 方法进行分块流式传输。以下示例使用了带有 `streaming=True` 的回调处理程序：

```python
local_path = (
    "./models/Meta-Llama-3-8B-Instruct.Q4_0.gguf"  # replace with your local file path
)
```

```python
from langchain_core.callbacks import BaseCallbackHandler

count = 0


class MyCustomHandler(BaseCallbackHandler):
    def on_llm_new_token(self, token: str, **kwargs) -> None:
        global count
        if count < 10:
            print(f"Token: {token}")
            count += 1


# Verbose is required to pass to the callback manager
llm = GPT4All(model=local_path, callbacks=[MyCustomHandler()], streaming=True)

# If you want to use a custom model add the backend parameter
# Check https://docs.gpt4all.io/gpt4all_python.html for supported backends
# llm = GPT4All(model=local_path, backend="gptj", callbacks=callbacks, streaming=True)

chain = prompt | llm

question = "What NFL team won the Super Bowl in the year Justin Bieber was born?"

# Streamed tokens will be logged/aggregated via the passed callback
res = chain.invoke({"question": question})
```
```output
Token:  Justin
Token:  Bieber
Token:  was
Token:  born
Token:  on
Token:  March
Token:  
Token: 1
Token: ,
Token:
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)