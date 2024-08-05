---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/retrievers/you-retriever.ipynb
---

# You.com

>[you.com API](https://api.you.com) 是一套工具，旨在帮助开发者将大型语言模型（LLMs）的输出与最新、最准确、最相关的信息结合起来，这些信息可能未包含在其训练数据集中。

## 设置

检索器位于 `langchain-community` 包中。

您还需要设置您的 you.com API 密钥。

```python
%pip install --upgrade --quiet langchain-community
```

```python
import os

os.environ["YDC_API_KEY"] = ""

# 用于链式调用部分
os.environ["OPENAI_API_KEY"] = ""

## 替代方案：从 .env 文件加载 YDC_API_KEY

# !pip install --quiet -U python-dotenv
# import dotenv
# dotenv.load_dotenv()
```

设置 [LangSmith](https://smith.langchain.com/) 以获得最佳的可观察性也是有帮助的（但不是必需的）。

```python
# os.environ["LANGCHAIN_TRACING_V2"] = "true"
# os.environ["LANGCHAIN_API_KEY"] = getpass.getpass()
# os.environ["LANGCHAIN_PROJECT"] = 'Experimentz'
```

## 实用工具使用


```python
from langchain_community.utilities import YouSearchAPIWrapper

utility = YouSearchAPIWrapper(num_web_results=1)

utility
```


```python
import json

# .raw_results 返回 API 的未修改响应
response = utility.raw_results(query="What is the weather in NY")
# API 返回一个包含 `hits` 键的对象，该键包含命中的列表
hits = response["hits"]

# 使用 `num_web_results=1`，`hits` 应该长度为 1
print(len(hits))

print(json.dumps(hits, indent=2))
```
```output
1
[
  {
    "description": "Be prepared with the most accurate 10-day forecast for Manhattan, NY with highs, lows, chance of precipitation from The Weather Channel and Weather.com",
    "snippets": [...
    ],
    "thumbnail_url": "https://imgs.search.brave.com/9xHc5-Bh2lvLyRJwQqeegm3gzoF6hawlpF8LZEjFLo8/rs:fit:200:200:1/g:ce/aHR0cHM6Ly9zLnct/eC5jby8yNDB4MTgw/X3R3Y19kZWZhdWx0/LnBuZw",
    "title": "10-Day Weather Forecast for Manhattan, NY - The Weather Channel ...",
    "url": "https://weather.com/weather/tenday/l/New+York+NY+USNY0996:1:US"
  }
]
```

```python
# .results 返回解析后的结果，每个片段都是一个 Document
response = utility.results(query="What is the weather in NY")

# .results 应该为每个 `snippet` 有一个 Document
print(len(response))

print(response)
```

## 检索器使用


```python
from langchain_community.retrievers.you import YouRetriever

retriever = YouRetriever(num_web_results=1)

retriever
```


```python
# .invoke wraps utility.results
response = retriever.invoke("What is the weather in NY")

# .invoke should have a Document for each `snippet`
print(len(response))

print(response)
```
```output
7
```

## 链接


```python
# you need a model to use in the chain
!pip install --upgrade --quiet langchain-openai
```


```python
from langchain_community.retrievers.you import YouRetriever
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI

# set up runnable
runnable = RunnablePassthrough

# set up retriever, limit sources to one
retriever = YouRetriever(num_web_results=1)

# set up model
model = ChatOpenAI(model="gpt-3.5-turbo-16k")

# set up output parser
output_parser = StrOutputParser()
```

### 调用


```python
# set up prompt that expects one question
prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the context provided.

Context: {context}

Question: {question}"""
)

# set up chain
chain = (
    runnable.assign(context=(lambda x: x["question"]) | retriever)
    | prompt
    | model
    | output_parser
)

output = chain.invoke({"question": "what is the weather in NY today"})

print(output)
```
```output
The weather in New York City today is 43° with a high/low of --/39°. The wind is 3 mph, humidity is 63%, and the air quality is considered good.
```

### 流

```python
# set up prompt that expects one question
prompt = ChatPromptTemplate.from_template(
    """Answer the question based only on the context provided.

Context: {context}

Question: {question}"""
)

# set up chain - same as above
chain = (
    runnable.assign(context=(lambda x: x["question"]) | retriever)
    | prompt
    | model
    | output_parser
)

for s in chain.stream({"question": "what is the weather in NY today"}):
    print(s, end="", flush=True)
```
```output
The weather in New York City today is a high of 39°F and a low of 31°F with a feels like temperature of 43°F. The wind speed is 3 mph, humidity is 63%, and the air quality is considered to be good.
```

### 批处理


```python
chain = (
    runnable.assign(context=(lambda x: x["question"]) | retriever)
    | prompt
    | model
    | output_parser
)

output = chain.batch(
    [
        {"question": "what is the weather in NY today"},
        {"question": "what is the weather in sf today"},
    ]
)

for o in output:
    print(o)
```
```output
Based on the provided context, the weather in New York City today is 43° with a high/low of --/39°.
Based on the provided context, the current weather in San Francisco is partly cloudy with a temperature of 61°F and a humidity of 57%.
```

## 相关

- Retriever [概念指南](/docs/concepts/#retrievers)
- Retriever [操作指南](/docs/how_to/#retrievers)