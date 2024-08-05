---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/serialization.ipynb
---

# 如何保存和加载 LangChain 对象

LangChain 类实现了标准的序列化方法。使用这些方法序列化 LangChain 对象有一些优势：

- 秘密信息，如 API 密钥，与其他参数分开，并可以在反序列化时重新加载到对象中；
- 反序列化在不同的包版本之间保持兼容，因此使用 LangChain 的一个版本序列化的对象可以在另一个版本中正确反序列化。

要使用此系统保存和加载 LangChain 对象，请使用 `dumpd`、`dumps`、`load` 和 `loads` 函数，这些函数位于 `langchain-core` 的 [load module](https://api.python.langchain.com/en/latest/core_api_reference.html#module-langchain_core.load)。这些函数支持 JSON 和 JSON 可序列化对象。

所有继承自 [Serializable](https://api.python.langchain.com/en/latest/load/langchain_core.load.serializable.Serializable.html) 的 LangChain 对象都是 JSON 可序列化的。示例包括 [messages](https://api.python.langchain.com/en/latest/core_api_reference.html#module-langchain_core.messages)、[document objects](https://api.python.langchain.com/en/latest/documents/langchain_core.documents.base.Document.html)（例如，从 [retrievers](/docs/concepts/#retrievers) 返回的对象）以及大多数通过 LangChain 表达式语言实现的 [Runnables](/docs/concepts/#langchain-expression-language-lcel)，如聊天模型、检索器和 [chains](/docs/how_to/sequence)。

下面我们通过一个简单的 [LLM chain](/docs/tutorials/llm_chain) 示例进行演示。

:::caution

使用 `load` 和 `loads` 进行反序列化可以实例化任何可序列化的 LangChain 对象。仅在信任的输入下使用此功能！

反序列化是一个 beta 功能，可能会有所更改。
:::


```python
from langchain_core.load import dumpd, dumps, load, loads
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", "Translate the following into {language}:"),
        ("user", "{text}"),
    ],
)

llm = ChatOpenAI(model="gpt-3.5-turbo-0125", api_key="llm-api-key")

chain = prompt | llm
```

## 保存对象

### 转换为 json


```python
string_representation = dumps(chain, pretty=True)
print(string_representation[:500])
```
```output
{
  "lc": 1,
  "type": "constructor",
  "id": [
    "langchain",
    "schema",
    "runnable",
    "RunnableSequence"
  ],
  "kwargs": {
    "first": {
      "lc": 1,
      "type": "constructor",
      "id": [
        "langchain",
        "prompts",
        "chat",
        "ChatPromptTemplate"
      ],
      "kwargs": {
        "input_variables": [
          "language",
          "text"
        ],
        "messages": [
          {
            "lc": 1,
            "type": "constructor",
```

### 转换为可序列化为 JSON 的 Python 字典


```python
dict_representation = dumpd(chain)

print(type(dict_representation))
```
```output
<class 'dict'>
```

### 保存到磁盘


```python
import json

with open("/tmp/chain.json", "w") as fp:
    json.dump(string_representation, fp)
```

请注意，API 密钥在序列化表示中被隐藏。被视为机密的参数由 LangChain 对象的 `.lc_secrets` 属性指定：


```python
chain.last.lc_secrets
```



```output
{'openai_api_key': 'OPENAI_API_KEY'}
```

## 加载对象

在 `load` 和 `loads` 中指定 `secrets_map` 将把相应的密钥加载到反序列化的 LangChain 对象上。

### 从字符串


```python
chain = loads(string_representation, secrets_map={"OPENAI_API_KEY": "llm-api-key"})
```

### 从字典


```python
chain = load(dict_representation, secrets_map={"OPENAI_API_KEY": "llm-api-key"})
```

### 从磁盘

```python
with open("/tmp/chain.json", "r") as fp:
    chain = loads(json.load(fp), secrets_map={"OPENAI_API_KEY": "llm-api-key"})
```

请注意，我们恢复了在本指南开始时指定的 API 密钥：

```python
chain.last.openai_api_key.get_secret_value()
```

```output
'llm-api-key'
```