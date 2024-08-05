---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/example_selectors_length_based.ipynb
---

# 如何按长度选择示例

此示例选择器根据长度选择使用哪些示例。当您担心构建一个超出上下文窗口长度的提示时，这非常有用。对于较长的输入，它将选择较少的示例进行包含，而对于较短的输入，它将选择更多的示例。

```python
from langchain_core.example_selectors import LengthBasedExampleSelector
from langchain_core.prompts import FewShotPromptTemplate, PromptTemplate

# 创建反义词的假任务示例。
examples = [
    {"input": "happy", "output": "sad"},
    {"input": "tall", "output": "short"},
    {"input": "energetic", "output": "lethargic"},
    {"input": "sunny", "output": "gloomy"},
    {"input": "windy", "output": "calm"},
]

example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="Input: {input}\nOutput: {output}",
)
example_selector = LengthBasedExampleSelector(
    # 可供选择的示例。
    examples=examples,
    # 用于格式化示例的PromptTemplate。
    example_prompt=example_prompt,
    # 格式化示例的最大长度。
    # 长度通过下面的get_text_length函数进行测量。
    max_length=25,
    # 用于获取字符串长度的函数，用于确定包含哪些示例。它被注释掉，因为
    # 如果没有指定，则提供为默认值。
    # get_text_length: Callable[[str], int] = lambda x: len(re.split("\n| ", x))
)
dynamic_prompt = FewShotPromptTemplate(
    # 我们提供一个ExampleSelector而不是示例。
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="给出每个输入的反义词",
    suffix="Input: {adjective}\nOutput:",
    input_variables=["adjective"],
)
```

```python
# 一个小输入的示例，因此它选择所有示例。
print(dynamic_prompt.format(adjective="big"))
```
```output
给出每个输入的反义词

Input: happy
Output: sad

Input: tall
Output: short

Input: energetic
Output: lethargic

Input: sunny
Output: gloomy

Input: windy
Output: calm

Input: big
Output:
```

```python
# 一个长输入的示例，因此它只选择一个示例。
long_string = "big and huge and massive and large and gigantic and tall and much much much much much bigger than everything else"
print(dynamic_prompt.format(adjective=long_string))
```
```output
给出每个输入的反义词

Input: happy
Output: sad

Input: big and huge and massive and large and gigantic and tall and much much much much much bigger than everything else
Output:
```

```python
# 您还可以向示例选择器添加示例。
new_example = {"input": "big", "output": "small"}
dynamic_prompt.example_selector.add_example(new_example)
print(dynamic_prompt.format(adjective="enthusiastic"))
```
```output
给出每个输入的反义词

Input: happy
Output: sad

Input: tall
Output: short

Input: energetic
Output: lethargic

Input: sunny
Output: gloomy

Input: windy
Output: calm

Input: big
Output: small

Input: enthusiastic
Output:
```