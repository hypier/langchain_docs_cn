---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/few_shot_examples.ipynb
sidebar_position: 3
---

# 如何使用少量示例

:::info 先决条件

本指南假设您熟悉以下概念：
- [提示模板](/docs/concepts/#prompt-templates)
- [示例选择器](/docs/concepts/#example-selectors)
- [LLMs](/docs/concepts/#llms)
- [向量存储](/docs/concepts/#vector-stores)

:::

在本指南中，我们将学习如何创建一个简单的提示模板，该模板在生成时为模型提供示例输入和输出。为LLM提供少量这样的示例称为少量学习（few-shotting），这是一种简单而强大的引导生成的方法，在某些情况下可以显著提高模型性能。

少量学习提示模板可以从一组示例构建，也可以从负责从定义的示例集中选择子集的[示例选择器](https://api.python.langchain.com/en/latest/example_selectors/langchain_core.example_selectors.base.BaseExampleSelector.html)类构建。

本指南将涵盖使用字符串提示模板的少量学习。有关使用聊天模型的聊天消息进行少量学习的指南，请参见[这里](/docs/how_to/few_shot_examples_chat/)。

## 为少量示例创建格式化器

配置一个格式化器，将少量示例格式化为字符串。该格式化器应为 `PromptTemplate` 对象。

```python
from langchain_core.prompts import PromptTemplate

example_prompt = PromptTemplate.from_template("Question: {question}\n{answer}")
```

## 创建示例集

接下来，我们将创建一个少量示例的列表。每个示例应该是一个字典，表示我们上面定义的格式化提示的示例输入。

```python
examples = [
    {
        "question": "谁活得更久，穆罕默德·阿里还是艾伦·图灵？",
        "answer": """
是否需要后续问题：是。
后续问题：穆罕默德·阿里去世时多大岁数？
中间答案：穆罕默德·阿里去世时74岁。
后续问题：艾伦·图灵去世时多大岁数？
中间答案：艾伦·图灵去世时41岁。
所以最终答案是：穆罕默德·阿里
""",
    },
    {
        "question": "craigslist的创始人出生于何时？",
        "answer": """
是否需要后续问题：是。
后续问题：谁是craigslist的创始人？
中间答案：craigslist是由克雷格·纽马克创立的。
后续问题：克雷格·纽马克出生于何时？
中间答案：克雷格·纽马克出生于1952年12月6日。
所以最终答案是：1952年12月6日
""",
    },
    {
        "question": "乔治·华盛顿的外祖父是谁？",
        "answer": """
是否需要后续问题：是。
后续问题：乔治·华盛顿的母亲是谁？
中间答案：乔治·华盛顿的母亲是玛丽·博尔·华盛顿。
后续问题：玛丽·博尔·华盛顿的父亲是谁？
中间答案：玛丽·博尔·华盛顿的父亲是约瑟夫·博尔。
所以最终答案是：约瑟夫·博尔
""",
    },
    {
        "question": "《大白鲨》和《皇家赌场》的导演来自同一个国家吗？",
        "answer": """
是否需要后续问题：是。
后续问题：《大白鲨》的导演是谁？
中间答案：《大白鲨》的导演是史蒂文·斯皮尔伯格。
后续问题：史蒂文·斯皮尔伯格来自哪里？
中间答案：美国。
后续问题：《皇家赌场》的导演是谁？
中间答案：《皇家赌场》的导演是马丁·坎贝尔。
后续问题：马丁·坎贝尔来自哪里？
中间答案：新西兰。
所以最终答案是：不
""",
    },
]
```

让我们用我们的一个示例测试格式化提示：

```python
print(example_prompt.invoke(examples[0]).to_string())
```
```output
问题：谁活得更久，穆罕默德·阿里还是艾伦·图灵？

是否需要后续问题：是。
后续问题：穆罕默德·阿里去世时多大岁数？
中间答案：穆罕默德·阿里去世时74岁。
后续问题：艾伦·图灵去世时多大岁数？
中间答案：艾伦·图灵去世时41岁。
所以最终答案是：穆罕默德·阿里
```

### 将示例和格式化器传递给 `FewShotPromptTemplate`

最后，创建一个 [`FewShotPromptTemplate`](https://api.python.langchain.com/en/latest/prompts/langchain_core.prompts.few_shot.FewShotPromptTemplate.html) 对象。该对象接受少量示例和少量示例的格式化器。当格式化此 `FewShotPromptTemplate` 时，它使用 `example_prompt` 格式化传递的示例，然后将它们添加到最终提示的 `suffix` 之前：

```python
from langchain_core.prompts import FewShotPromptTemplate

prompt = FewShotPromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
    suffix="Question: {input}",
    input_variables=["input"],
)

print(
    prompt.invoke({"input": "Who was the father of Mary Ball Washington?"}).to_string()
)
```
```output
Question: Who lived longer, Muhammad Ali or Alan Turing?

Are follow up questions needed here: Yes.
Follow up: How old was Muhammad Ali when he died?
Intermediate answer: Muhammad Ali was 74 years old when he died.
Follow up: How old was Alan Turing when he died?
Intermediate answer: Alan Turing was 41 years old when he died.
So the final answer is: Muhammad Ali


Question: When was the founder of craigslist born?

Are follow up questions needed here: Yes.
Follow up: Who was the founder of craigslist?
Intermediate answer: Craigslist was founded by Craig Newmark.
Follow up: When was Craig Newmark born?
Intermediate answer: Craig Newmark was born on December 6, 1952.
So the final answer is: December 6, 1952


Question: Who was the maternal grandfather of George Washington?

Are follow up questions needed here: Yes.
Follow up: Who was the mother of George Washington?
Intermediate answer: The mother of George Washington was Mary Ball Washington.
Follow up: Who was the father of Mary Ball Washington?
Intermediate answer: The father of Mary Ball Washington was Joseph Ball.
So the final answer is: Joseph Ball


Question: Are both the directors of Jaws and Casino Royale from the same country?

Are follow up questions needed here: Yes.
Follow up: Who is the director of Jaws?
Intermediate Answer: The director of Jaws is Steven Spielberg.
Follow up: Where is Steven Spielberg from?
Intermediate Answer: The United States.
Follow up: Who is the director of Casino Royale?
Intermediate Answer: The director of Casino Royale is Martin Campbell.
Follow up: Where is Martin Campbell from?
Intermediate Answer: New Zealand.
So the final answer is: No


Question: Who was the father of Mary Ball Washington?
```
通过向模型提供这样的示例，我们可以引导模型更好地响应。

## 使用示例选择器

我们将重用上一节中的示例集和格式化器。然而，我们将把示例输入到一个名为 [`SemanticSimilarityExampleSelector`](https://api.python.langchain.com/en/latest/example_selectors/langchain_core.example_selectors.semantic_similarity.SemanticSimilarityExampleSelector.html) 的 `ExampleSelector` 实现中，而不是直接输入到 `FewShotPromptTemplate` 对象中。该类根据输入与示例之间的相似性，从初始集合中选择少量示例。它使用嵌入模型计算输入与少量示例之间的相似性，并使用向量存储执行最近邻搜索。

为了展示它的样子，让我们初始化一个实例并单独调用它：

```python
from langchain_chroma import Chroma
from langchain_core.example_selectors import SemanticSimilarityExampleSelector
from langchain_openai import OpenAIEmbeddings

example_selector = SemanticSimilarityExampleSelector.from_examples(
    # This is the list of examples available to select from.
    examples,
    # This is the embedding class used to produce embeddings which are used to measure semantic similarity.
    OpenAIEmbeddings(),
    # This is the VectorStore class that is used to store the embeddings and do a similarity search over.
    Chroma,
    # This is the number of examples to produce.
    k=1,
)

# Select the most similar example to the input.
question = "Who was the father of Mary Ball Washington?"
selected_examples = example_selector.select_examples({"question": question})
print(f"Examples most similar to the input: {question}")
for example in selected_examples:
    print("\n")
    for k, v in example.items():
        print(f"{k}: {v}")
```
```output
Examples most similar to the input: Who was the father of Mary Ball Washington?


answer: 
Are follow up questions needed here: Yes.
Follow up: Who was the mother of George Washington?
Intermediate answer: The mother of George Washington was Mary Ball Washington.
Follow up: Who was the father of Mary Ball Washington?
Intermediate answer: The father of Mary Ball Washington was Joseph Ball.
So the final answer is: Joseph Ball

question: Who was the maternal grandfather of George Washington?
```
现在，让我们创建一个 `FewShotPromptTemplate` 对象。该对象接受示例选择器和少量示例的格式化提示。

```python
prompt = FewShotPromptTemplate(
    example_selector=example_selector,
    example_prompt=example_prompt,
    suffix="Question: {input}",
    input_variables=["input"],
)

print(
    prompt.invoke({"input": "Who was the father of Mary Ball Washington?"}).to_string()
)
```
```output
Question: Who was the maternal grandfather of George Washington?

Are follow up questions needed here: Yes.
Follow up: Who was the mother of George Washington?
Intermediate answer: The mother of George Washington was Mary Ball Washington.
Follow up: Who was the father of Mary Ball Washington?
Intermediate answer: The father of Mary Ball Washington was Joseph Ball.
So the final answer is: Joseph Ball


Question: Who was the father of Mary Ball Washington?
```

## 下一步

您现在已经学习了如何在提示中添加少量示例。

接下来，请查看本节中有关提示模板的其他操作指南，相关的关于[与聊天模型进行少量示例的操作指南](/docs/how_to/few_shot_examples_chat)，或其他[示例选择器操作指南](/docs/how_to/example_selectors/)。