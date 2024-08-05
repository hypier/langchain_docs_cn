---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/ipex_llm.ipynb
---

# IPEX-LLM

> [IPEX-LLM](https://github.com/intel-analytics/ipex-llm/) 是一个用于在 Intel CPU 和 GPU（例如，带有 iGPU 的本地 PC、离散 GPU，如 Arc、Flex 和 Max）上以非常低的延迟运行 LLM 的 PyTorch 库。

本示例介绍如何使用 LangChain 与 `ipex-llm` 进行文本生成的交互。

## 设置


```python
# 更新 Langchain

%pip install -qU langchain langchain-community
```

安装 IEPX-LLM 以在 Intel CPU 上本地运行 LLM。


```python
%pip install --pre --upgrade ipex-llm[all]
```

## 基本用法


```python
import warnings

from langchain.chains import LLMChain
from langchain_community.llms import IpexLLM
from langchain_core.prompts import PromptTemplate

warnings.filterwarnings("ignore", category=UserWarning, message=".*padding_mask.*")
```

为您的模型指定提示模板。在此示例中，我们使用 [vicuna-1.5](https://huggingface.co/lmsys/vicuna-7b-v1.5) 模型。如果您使用的是不同的模型，请相应选择合适的模板。


```python
template = "USER: {question}\nASSISTANT:"
prompt = PromptTemplate(template=template, input_variables=["question"])
```

使用 `IpexLLM.from_model_id` 在本地加载模型。它将直接以 Huggingface 格式加载模型，并自动转换为低位格式以进行推理。


```python
llm = IpexLLM.from_model_id(
    model_id="lmsys/vicuna-7b-v1.5",
    model_kwargs={"temperature": 0, "max_length": 64, "trust_remote_code": True},
)
```

在链中使用它：


```python
llm_chain = prompt | llm

question = "What is AI?"
output = llm_chain.invoke(question)
```

## 保存/加载低位模型
另外，您可以将低位模型保存到磁盘一次，然后使用 `from_model_id_low_bit` 代替 `from_model_id` 来重新加载它以供后续使用——即使在不同的机器之间。这是节省空间的，因为低位模型所需的磁盘空间显著少于原始模型。而且 `from_model_id_low_bit` 在速度和内存使用方面也比 `from_model_id` 更高效，因为它跳过了模型转换步骤。

要保存低位模型，请使用 `save_low_bit`，如下所示。

```python
saved_lowbit_model_path = "./vicuna-7b-1.5-low-bit"  # 保存低位模型的路径
llm.model.save_low_bit(saved_lowbit_model_path)
del llm
```

从保存的低位模型路径加载模型，如下所示。
> 请注意，低位模型的保存路径仅包括模型本身，而不包括分词器。如果您希望将所有内容放在一个地方，您需要手动从原始模型的目录下载或复制分词器文件到低位模型保存的位置。

```python
llm_lowbit = IpexLLM.from_model_id_low_bit(
    model_id=saved_lowbit_model_path,
    tokenizer_id="lmsys/vicuna-7b-v1.5",
    # tokenizer_name=saved_lowbit_model_path,  # 如果您希望以这种方式使用它，请将分词器复制到保存路径
    model_kwargs={"temperature": 0, "max_length": 64, "trust_remote_code": True},
)
```

在链中使用加载的模型：

```python
llm_chain = prompt | llm_lowbit

question = "What is AI?"
output = llm_chain.invoke(question)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)