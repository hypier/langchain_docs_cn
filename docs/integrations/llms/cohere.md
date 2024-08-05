---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/cohere.ipynb
---

# Cohere

:::caution
您当前正在查看文档，介绍了如何使用 Cohere 模型作为 [文本补全模型](/docs/concepts/#llms)。许多流行的 Cohere 模型是 [聊天补全模型](/docs/concepts/#chat-models)。

您可能想查看 [这页](/docs/integrations/chat/cohere/)。
:::

>[Cohere](https://cohere.ai/about) 是一家加拿大初创公司，提供自然语言处理模型，帮助公司改善人机交互。

前往 [API 参考](https://api.python.langchain.com/en/latest/llms/langchain_community.llms.cohere.Cohere.html) 获取所有属性和方法的详细文档。

## 设置

集成位于 `langchain-community` 包中。我们还需要安装 `cohere` 包。我们可以通过以下命令安装这些包：

```bash
pip install -U langchain-community langchain-cohere
```

我们还需要获取一个 [Cohere API 密钥](https://cohere.com/) 并设置 `COHERE_API_KEY` 环境变量：

```python
import getpass
import os

os.environ["COHERE_API_KEY"] = getpass.getpass()
```
```output
 ········
```
设置 [LangSmith](https://smith.langchain.com/) 以获得最佳的可观察性也是有帮助的（但不是必需的）

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
```

## 用法

Cohere 支持所有 [LLM](/docs/how_to#llms) 功能：


```python
from langchain_cohere import Cohere
from langchain_core.messages import HumanMessage
```


```python
model = Cohere(max_tokens=256, temperature=0.75)
```


```python
message = "Knock knock"
model.invoke(message)
```



```output
" Who's there?"
```



```python
await model.ainvoke(message)
```



```output
" Who's there?"
```



```python
for chunk in model.stream(message):
    print(chunk, end="", flush=True)
```
```output
 Who's there?
```

```python
model.batch([message])
```



```output
[" Who's there?"]
```


您还可以轻松地与提示模板结合，以便于结构化用户输入。我们可以使用 [LCEL](/docs/concepts#langchain-expression-language-lcel) 来实现这一点。


```python
from langchain_core.prompts import PromptTemplate

prompt = PromptTemplate.from_template("Tell me a joke about {topic}")
chain = prompt | model
```


```python
chain.invoke({"topic": "bears"})
```



```output
' Why did the teddy bear cross the road?\nBecause he had bear crossings.\n\nWould you like to hear another joke? '
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)