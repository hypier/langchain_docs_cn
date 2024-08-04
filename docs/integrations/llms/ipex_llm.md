---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/ipex_llm.ipynb
---
# IPEX-LLM

> [IPEX-LLM](https://github.com/intel-analytics/ipex-llm/) is a PyTorch library for running LLM on Intel CPU and GPU (e.g., local PC with iGPU, discrete GPU such as Arc, Flex and Max) with very low latency. 

This example goes over how to use LangChain to interact with `ipex-llm` for text generation. 


## Setup


```python
# Update Langchain

%pip install -qU langchain langchain-community
```

Install IEPX-LLM for running LLMs locally on Intel CPU.


```python
%pip install --pre --upgrade ipex-llm[all]
```

## Basic Usage


```python
import warnings

from langchain.chains import LLMChain
from langchain_community.llms import IpexLLM
from langchain_core.prompts import PromptTemplate

warnings.filterwarnings("ignore", category=UserWarning, message=".*padding_mask.*")
```

Specify the prompt template for your model. In this example, we use the [vicuna-1.5](https://huggingface.co/lmsys/vicuna-7b-v1.5) model. If you're working with a different model, choose a proper template accordingly.


```python
template = "USER: {question}\nASSISTANT:"
prompt = PromptTemplate(template=template, input_variables=["question"])
```

Load the model locally using IpexLLM using `IpexLLM.from_model_id`. It will load the model directly in its Huggingface format and convert it automatically to low-bit format for inference.


```python
llm = IpexLLM.from_model_id(
    model_id="lmsys/vicuna-7b-v1.5",
    model_kwargs={"temperature": 0, "max_length": 64, "trust_remote_code": True},
)
```

Use it in Chains:


```python
llm_chain = prompt | llm

question = "What is AI?"
output = llm_chain.invoke(question)
```

## Save/Load Low-bit Model
Alternatively, you might save the low-bit model to disk once and use `from_model_id_low_bit` instead of `from_model_id` to reload it for later use - even across different machines. It is space-efficient, as the low-bit model demands significantly less disk space than the original model. And `from_model_id_low_bit` is also more efficient than `from_model_id` in terms of speed and memory usage, as it skips the model conversion step.

To save the low-bit model, use `save_low_bit` as follows.


```python
saved_lowbit_model_path = "./vicuna-7b-1.5-low-bit"  # path to save low-bit model
llm.model.save_low_bit(saved_lowbit_model_path)
del llm
```

Load the model from saved lowbit model path as follows. 
> Note that the saved path for the low-bit model only includes the model itself but not the tokenizers. If you wish to have everything in one place, you will need to manually download or copy the tokenizer files from the original model's directory to the location where the low-bit model is saved.


```python
llm_lowbit = IpexLLM.from_model_id_low_bit(
    model_id=saved_lowbit_model_path,
    tokenizer_id="lmsys/vicuna-7b-v1.5",
    # tokenizer_name=saved_lowbit_model_path,  # copy the tokenizers to saved path if you want to use it this way
    model_kwargs={"temperature": 0, "max_length": 64, "trust_remote_code": True},
)
```

Use the loaded model in Chains:


```python
llm_chain = prompt | llm_lowbit


question = "What is AI?"
output = llm_chain.invoke(question)
```


## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)
