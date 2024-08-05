---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/example_selectors_ngram.ipynb
---

# 如何通过 n-gram 重叠选择示例

`NGramOverlapExampleSelector` 根据与输入的相似性选择和排序示例，依据 ngram 重叠得分。ngram 重叠得分是一个介于 0.0 和 1.0 之间的浮动值（包括 0.0 和 1.0）。

选择器允许设置一个阈值得分。ngram 重叠得分小于或等于阈值的示例将被排除。默认情况下，阈值设置为 -1.0，因此不会排除任何示例，只会对它们重新排序。将阈值设置为 0.0 将排除与输入没有 ngram 重叠的示例。



```python
from langchain_community.example_selectors import NGramOverlapExampleSelector
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate

example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="Input: {input}\nOutput: {output}",
)

# 虚构翻译任务的示例。
examples = [
    {"input": "See Spot run.", "output": "Ver correr a Spot."},
    {"input": "My dog barks.", "output": "Mi perro ladra."},
    {"input": "Spot can run.", "output": "Spot puede correr."},
]
```


```python
example_selector = NGramOverlapExampleSelector(
    # 可供选择的示例。
    examples=examples,
    # 用于格式化示例的 PromptTemplate。
    example_prompt=example_prompt,
    # 选择器停止的阈值。
    # 默认设置为 -1.0。
    threshold=-1.0,
    # 对于负阈值：
    # 选择器按 ngram 重叠得分对示例进行排序，不排除任何示例。
    # 对于大于 1.0 的阈值：
    # 选择器排除所有示例，并返回一个空列表。
    # 对于等于 0.0 的阈值：
    # 选择器按 ngram 重叠得分对示例进行排序，
    # 并排除与输入没有 ngram 重叠的示例。
)
dynamic_prompt = FewShotPromptTemplate(
    # 我们提供一个 ExampleSelector 而不是示例。
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="给每个输入提供西班牙语翻译",
    suffix="Input: {sentence}\nOutput:",
    input_variables=["sentence"],
)
```


```python
# 一个与“Spot can run.”有较大 ngram 重叠且与“My dog barks.”没有重叠的示例输入。
print(dynamic_prompt.format(sentence="Spot can run fast."))
```
```output
给每个输入提供西班牙语翻译

Input: Spot can run.
Output: Spot puede correr.

Input: See Spot run.
Output: Ver correr a Spot.

Input: My dog barks.
Output: Mi perro ladra.

Input: Spot can run fast.
Output:
```

```python
# 您也可以向 NGramOverlapExampleSelector 添加示例。
new_example = {"input": "Spot plays fetch.", "output": "Spot juega a buscar."}

example_selector.add_example(new_example)
print(dynamic_prompt.format(sentence="Spot can run fast."))
```
```output
给每个输入提供西班牙语翻译

Input: Spot can run.
Output: Spot puede correr.

Input: See Spot run.
Output: Ver correr a Spot.

Input: Spot plays fetch.
Output: Spot juega a buscar.

Input: My dog barks.
Output: Mi perro ladra.

Input: Spot can run fast.
Output:
```

```python
# 您可以设置一个阈值，以排除示例。
# 例如，将阈值设置为 0.0 将排除与输入没有 ngram 重叠的示例。
# 由于“My dog barks.”与“Spot can run fast.”没有 ngram 重叠，
# 因此被排除。
example_selector.threshold = 0.0
print(dynamic_prompt.format(sentence="Spot can run fast."))
```
```output
给每个输入提供西班牙语翻译

Input: Spot can run.
Output: Spot puede correr.

Input: See Spot run.
Output: Ver correr a Spot.

Input: Spot plays fetch.
Output: Spot juega a buscar.

Input: Spot can run fast.
Output:
```

```python
# 设置小的非零阈值
example_selector.threshold = 0.09
print(dynamic_prompt.format(sentence="Spot can play fetch."))
```
```output
给每个输入提供西班牙语翻译

Input: Spot can run.
Output: Spot puede correr.

Input: Spot plays fetch.
Output: Spot juega a buscar.

Input: Spot can play fetch.
Output:
```

```python
# 设置大于 1.0 的阈值
example_selector.threshold = 1.0 + 1e-9
print(dynamic_prompt.format(sentence="Spot can play fetch."))
```
```output
给每个输入提供西班牙语翻译

Input: Spot can play fetch.
Output:
```