---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/weight_only_quantization.ipynb
---

# Intel 仅权重量化

## 仅权重量化的 Huggingface 模型与 Intel Extension for Transformers 管道

Hugging Face 模型可以通过 `WeightOnlyQuantPipeline` 类在本地运行，仅进行权重量化。

[Hugging Face 模型库](https://huggingface.co/models) 提供了超过 12 万个模型、2 万个数据集和 5 万个演示应用（Spaces），所有这些都是开源和公开可用的，用户可以在一个在线平台上轻松协作，共同构建机器学习。

这些可以通过这个本地管道封装类从 LangChain 调用。

要使用此功能，您需要安装 ``transformers`` python [包](https://pypi.org/project/transformers/)，以及 [pytorch](https://pytorch.org/get-started/locally/) 和 [intel-extension-for-transformers](https://github.com/intel/intel-extension-for-transformers)。

```python
%pip install transformers --quiet
%pip install intel-extension-for-transformers
```

### 模型加载

可以通过使用 `from_model_id` 方法指定模型参数来加载模型。模型参数包括 intel_extension_for_transformers 中的 `WeightOnlyQuantConfig` 类。

```python
from intel_extension_for_transformers.transformers import WeightOnlyQuantConfig
from langchain_community.llms.weight_only_quantization import WeightOnlyQuantPipeline

conf = WeightOnlyQuantConfig(weight_dtype="nf4")
hf = WeightOnlyQuantPipeline.from_model_id(
    model_id="google/flan-t5-large",
    task="text2text-generation",
    quantization_config=conf,
    pipeline_kwargs={"max_new_tokens": 10},
)
```

也可以通过直接传入现有的 `transformers` 管道来加载它们。

```python
from intel_extension_for_transformers.transformers import AutoModelForSeq2SeqLM
from transformers import AutoTokenizer, pipeline

model_id = "google/flan-t5-large"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForSeq2SeqLM.from_pretrained(model_id)
pipe = pipeline(
    "text2text-generation", model=model, tokenizer=tokenizer, max_new_tokens=10
)
hf = WeightOnlyQuantPipeline(pipeline=pipe)
```

### 创建链

将模型加载到内存中后，可以用提示词将其组合形成链。

```python
from langchain_core.prompts import PromptTemplate

template = """Question: {question}

Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)

chain = prompt | hf

question = "What is electroencephalography?"

print(chain.invoke({"question": question}))
```

### CPU推理

现在intel-extension-for-transformers仅支持CPU设备推理。将很快支持intel GPU。在CPU机器上运行时，可以指定`device="cpu"`或`device=-1`参数将模型放置在CPU设备上。默认值为`-1`用于CPU推理。


```python
conf = WeightOnlyQuantConfig(weight_dtype="nf4")
llm = WeightOnlyQuantPipeline.from_model_id(
    model_id="google/flan-t5-large",
    task="text2text-generation",
    quantization_config=conf,
    pipeline_kwargs={"max_new_tokens": 10},
)

template = """Question: {question}

Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)

chain = prompt | llm

question = "What is electroencephalography?"

print(chain.invoke({"question": question}))
```

### 批量 CPU 推理

您还可以以批量模式在 CPU 上运行推理。

```python
conf = WeightOnlyQuantConfig(weight_dtype="nf4")
llm = WeightOnlyQuantPipeline.from_model_id(
    model_id="google/flan-t5-large",
    task="text2text-generation",
    quantization_config=conf,
    pipeline_kwargs={"max_new_tokens": 10},
)

chain = prompt | llm.bind(stop=["\n\n"])

questions = []
for i in range(4):
    questions.append({"question": f"What is the number {i} in french?"})

answers = chain.batch(questions)
for answer in answers:
    print(answer)
```

### Intel-extension-for-transformers 支持的数据类型

我们支持将权重量化为以下数据类型以进行存储（weight_dtype 在 WeightOnlyQuantConfig 中）：

* **int8**: 使用 8 位数据类型。
* **int4_fullrange**: 使用 int4 范围的 -8 值，与正常的 int4 范围 [-7,7] 相比。
* **int4_clip**: 裁剪并保留 int4 范围内的值，将其他值设置为零。
* **nf4**: 使用归一化的 4 位浮点数据类型。
* **fp4_e2m1**: 使用常规的 4 位浮点数据类型。“e2”表示 2 位用于指数，“m1”表示 1 位用于尾数。

虽然这些技术将权重存储为 4 或 8 位，但计算仍然在 float32、bfloat16 或 int8 中进行（compute_dtype 在 WeightOnlyQuantConfig 中）：
* **fp32**: 使用 float32 数据类型进行计算。
* **bf16**: 使用 bfloat16 数据类型进行计算。
* **int8**: 使用 8 位数据类型进行计算。

### 支持的算法矩阵

在 intel-extension-for-transformers 中支持的量化算法（WeightOnlyQuantConfig 中的算法）：

| 算法         |   PyTorch  |    LLM Runtime    |
|:--------------:|:----------:|:----------:|
|       RTN      |  &#10004;  |  &#10004;  |
|       AWQ      |  &#10004;  | 敬请期待 |
|      TEQ      | &#10004; | 敬请期待 |
> **RTN:** 一种我们可以非常直观地理解的量化方法。它不需要额外的数据集，是一种非常快速的量化方法。一般来说，RTN 将权重转换为均匀分布的整数数据类型，但一些算法，如 Qlora，提出了一种非均匀的 NF4 数据类型，并证明了其理论最优性。

> **AWQ:** 证明仅保护 1% 的显著权重可以大大减少量化误差。显著权重通道是通过观察每个通道的激活和权重分布来选择的。显著权重在量化前还会在乘以一个大比例因子后进行量化以进行保留。

> **TEQ:** 一种可训练的等效变换，在仅权重量化中保留 FP32 精度。它受到 AWQ 的启发，同时提供了一种新的解决方案，以搜索激活和权重之间的最佳通道缩放因子。

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)