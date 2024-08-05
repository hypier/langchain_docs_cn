---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/example_selectors_similarity.ipynb
---

# 如何通过相似性选择示例

该对象根据与输入的相似性选择示例。它通过找到与输入具有最大余弦相似度的嵌入示例来实现这一点。



```python
from langchain_chroma import Chroma
from langchain_core.example_selectors import SemanticSimilarityExampleSelector
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate
from langchain_openai import OpenAIEmbeddings

example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="Input: {input}\nOutput: {output}",
)

# 创建反义词的虚构任务示例。
examples = [
    {"input": "happy", "output": "sad"},
    {"input": "tall", "output": "short"},
    {"input": "energetic", "output": "lethargic"},
    {"input": "sunny", "output": "gloomy"},
    {"input": "windy", "output": "calm"},
]
```


```python
example_selector = SemanticSimilarityExampleSelector.from_examples(
    # 可供选择的示例列表。
    examples,
    # 用于生成嵌入的嵌入类，这些嵌入用于测量语义相似性。
    OpenAIEmbeddings(),
    # 用于存储嵌入并进行相似性搜索的 VectorStore 类。
    Chroma,
    # 生成的示例数量。
    k=1,
)
similar_prompt = FewShotPromptTemplate(
    # 我们提供一个 ExampleSelector，而不是示例。
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="给出每个输入的反义词",
    suffix="Input: {adjective}\nOutput:",
    input_variables=["adjective"],
)
```


```python
# 输入是情感，因此应选择 happy/sad 示例
print(similar_prompt.format(adjective="worried"))
```
```output
给出每个输入的反义词

Input: happy
Output: sad

Input: worried
Output:
```

```python
# 输入是一个测量，因此应选择 tall/short 示例
print(similar_prompt.format(adjective="large"))
```
```output
给出每个输入的反义词

Input: tall
Output: short

Input: large
Output:
```

```python
# 您也可以向 SemanticSimilarityExampleSelector 添加新示例
similar_prompt.example_selector.add_example(
    {"input": "enthusiastic", "output": "apathetic"}
)
print(similar_prompt.format(adjective="passionate"))
```
```output
给出每个输入的反义词

Input: enthusiastic
Output: apathetic

Input: passionate
Output:
```