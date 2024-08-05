---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/example_selectors_mmr.ipynb
---

# 如何通过最大边际相关性 (MMR) 选择示例

`MaxMarginalRelevanceExampleSelector` 根据与输入最相似的示例以及优化多样性的组合来选择示例。它通过找到与输入具有最大余弦相似度的示例的嵌入，并在此基础上迭代添加这些示例，同时对与已选择示例的接近程度进行惩罚。

```python
from langchain_community.vectorstores import FAISS
from langchain_core.example_selectors import (
    MaxMarginalRelevanceExampleSelector,
    SemanticSimilarityExampleSelector,
)
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate
from langchain_openai import OpenAIEmbeddings

example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="Input: {input}\nOutput: {output}",
)

# 创建反义词的假设任务示例
examples = [
    {"input": "happy", "output": "sad"},
    {"input": "tall", "output": "short"},
    {"input": "energetic", "output": "lethargic"},
    {"input": "sunny", "output": "gloomy"},
    {"input": "windy", "output": "calm"},
]
```

```python
example_selector = MaxMarginalRelevanceExampleSelector.from_examples(
    # 可供选择的示例列表。
    examples,
    # 用于生成嵌入的嵌入类，用于测量语义相似性。
    OpenAIEmbeddings(),
    # 用于存储嵌入并进行相似性搜索的 VectorStore 类。
    FAISS,
    # 要生成的示例数量。
    k=2,
)
mmr_prompt = FewShotPromptTemplate(
    # 我们提供一个 ExampleSelector 而不是示例。
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="给出每个输入的反义词",
    suffix="Input: {adjective}\nOutput:",
    input_variables=["adjective"],
)
```

```python
# 输入是一个感觉，因此应选择 happy/sad 示例作为第一个
print(mmr_prompt.format(adjective="worried"))
```
```output
给出每个输入的反义词

Input: happy
Output: sad

Input: windy
Output: calm

Input: worried
Output:
```

```python
# 让我们将其与仅基于相似性得到的结果进行比较，
# 通过使用 SemanticSimilarityExampleSelector 而不是 MaxMarginalRelevanceExampleSelector。
example_selector = SemanticSimilarityExampleSelector.from_examples(
    # 可供选择的示例列表。
    examples,
    # 用于生成嵌入的嵌入类，用于测量语义相似性。
    OpenAIEmbeddings(),
    # 用于存储嵌入并进行相似性搜索的 VectorStore 类。
    FAISS,
    # 要生成的示例数量。
    k=2,
)
similar_prompt = FewShotPromptTemplate(
    # 我们提供一个 ExampleSelector 而不是示例。
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="给出每个输入的反义词",
    suffix="Input: {adjective}\nOutput:",
    input_variables=["adjective"],
)
print(similar_prompt.format(adjective="worried"))
```
```output
给出每个输入的反义词

Input: happy
Output: sad

Input: sunny
Output: gloomy

Input: worried
Output:
```