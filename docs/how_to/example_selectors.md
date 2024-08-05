---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/example_selectors.ipynb
sidebar_position: 1
---

# 如何使用示例选择器

如果您有大量示例，您可能需要选择哪些示例包含在提示中。示例选择器是负责执行此操作的类。

基本接口定义如下：

```python
class BaseExampleSelector(ABC):
    """Interface for selecting examples to include in prompts."""

    @abstractmethod
    def select_examples(self, input_variables: Dict[str, str]) -> List[dict]:
        """Select which examples to use based on the inputs."""
        
    @abstractmethod
    def add_example(self, example: Dict[str, str]) -> Any:
        """Add new example to store."""
```

它需要定义的唯一方法是 ``select_examples`` 方法。该方法接受输入变量，然后返回示例列表。具体实现如何选择这些示例则由每个具体实现决定。

LangChain 有几种不同类型的示例选择器。有关所有这些类型的概述，请参见下表。

在本指南中，我们将介绍如何创建自定义示例选择器。

## 示例

为了使用示例选择器，我们需要创建一个示例列表。这些通常应该是示例输入和输出。为了演示的目的，让我们想象一下我们正在选择如何将英语翻译成意大利语的示例。

```python
examples = [
    {"input": "hi", "output": "ciao"},
    {"input": "bye", "output": "arrivederci"},
    {"input": "soccer", "output": "calcio"},
]
```

## 自定义示例选择器

让我们编写一个示例选择器，根据单词的长度选择要挑选的示例。


```python
from langchain_core.example_selectors.base import BaseExampleSelector


class CustomExampleSelector(BaseExampleSelector):
    def __init__(self, examples):
        self.examples = examples

    def add_example(self, example):
        self.examples.append(example)

    def select_examples(self, input_variables):
        # 这假设输入的一部分将是一个 'text' 键
        new_word = input_variables["input"]
        new_word_length = len(new_word)

        # 初始化变量以存储最佳匹配及其长度差
        best_match = None
        smallest_diff = float("inf")

        # 遍历每个示例
        for example in self.examples:
            # 计算当前示例第一个单词的长度差
            current_diff = abs(len(example["input"]) - new_word_length)

            # 如果当前示例在长度上更接近，则更新最佳匹配
            if current_diff < smallest_diff:
                smallest_diff = current_diff
                best_match = example

        return [best_match]
```


```python
example_selector = CustomExampleSelector(examples)
```


```python
example_selector.select_examples({"input": "okay"})
```



```output
[{'input': 'bye', 'output': 'arrivederci'}]
```



```python
example_selector.add_example({"input": "hand", "output": "mano"})
```


```python
example_selector.select_examples({"input": "okay"})
```



```output
[{'input': 'hand', 'output': 'mano'}]
```

## 用于提示

我们现在可以在提示中使用这个示例选择器


```python
from langchain_core.prompts.few_shot import FewShotPromptTemplate
from langchain_core.prompts.prompt import PromptTemplate

example_prompt = PromptTemplate.from_template("Input: {input} -> Output: {output}")
```


```python
prompt = FewShotPromptTemplate(
    example_selector=example_selector,
    example_prompt=example_prompt,
    suffix="Input: {input} -> Output:",
    prefix="将以下单词从英语翻译成意大利语：",
    input_variables=["input"],
)

print(prompt.format(input="word"))
```
```output
将以下单词从英语翻译成意大利语：

Input: hand -> Output: mano

Input: word -> Output:
```

## 示例选择器类型

| 名称       | 描述                                                                                       |
|------------|---------------------------------------------------------------------------------------------|
| 相似性     | 使用输入与示例之间的语义相似性来决定选择哪些示例。                                          |
| MMR        | 使用输入与示例之间的最大边际相关性来决定选择哪些示例。                                     |
| 长度       | 根据可以适应特定长度的示例数量来选择示例。                                                  |
| Ngram      | 使用输入与示例之间的n-gram重叠来决定选择哪些示例。                                         |