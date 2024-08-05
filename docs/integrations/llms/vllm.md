---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/vllm.ipynb
---

# vLLM

[vLLM](https://vllm.readthedocs.io/en/latest/index.html) 是一个快速且易于使用的 LLM 推理和服务库，提供：

* 最先进的服务吞吐量
* 使用 PagedAttention 高效管理注意力键和值内存
* 持续批处理传入请求
* 优化的 CUDA 内核

本笔记本介绍如何使用 langchain 和 vLLM 进行 LLM。

使用前，请确保已安装 `vllm` python 包。


```python
%pip install --upgrade --quiet  vllm -q
```


```python
from langchain_community.llms import VLLM

llm = VLLM(
    model="mosaicml/mpt-7b",
    trust_remote_code=True,  # mandatory for hf models
    max_new_tokens=128,
    top_k=10,
    top_p=0.95,
    temperature=0.8,
)

print(llm.invoke("What is the capital of France ?"))
```
```output
INFO 08-06 11:37:33 llm_engine.py:70] Initializing an LLM engine with config: model='mosaicml/mpt-7b', tokenizer='mosaicml/mpt-7b', tokenizer_mode=auto, trust_remote_code=True, dtype=torch.bfloat16, use_dummy_weights=False, download_dir=None, use_np_weights=False, tensor_parallel_size=1, seed=0)
INFO 08-06 11:37:41 llm_engine.py:196] # GPU blocks: 861, # CPU blocks: 512
``````output
Processed prompts: 100%|██████████| 1/1 [00:00<00:00,  2.00it/s]
``````output

What is the capital of France ? The capital of France is Paris.
``````output

```

## 在LLMChain中集成模型


```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate

template = """Question: {question}

Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)

llm_chain = LLMChain(prompt=prompt, llm=llm)

question = "Who was the US president in the year the first Pokemon game was released?"

print(llm_chain.invoke(question))
```
```output
Processed prompts: 100%|██████████| 1/1 [00:01<00:00,  1.34s/it]
``````output


1. 第一款Pokemon游戏于1996年发布。
2. 总统是比尔·克林顿。
3. 克林顿在1993年至2001年期间担任总统。
4. 答案是克林顿。
``````output

```

## 分布式推理

vLLM 支持分布式张量并行推理和服务。

要使用 LLM 类运行多 GPU 推理，请将 `tensor_parallel_size` 参数设置为您想要使用的 GPU 数量。例如，要在 4 个 GPU 上运行推理


```python
from langchain_community.llms import VLLM

llm = VLLM(
    model="mosaicml/mpt-30b",
    tensor_parallel_size=4,
    trust_remote_code=True,  # mandatory for hf models
)

llm.invoke("What is the future of AI?")
```

## 量化

vLLM 支持 `awq` 量化。要启用它，请将 `quantization` 传递给 `vllm_kwargs`。


```python
llm_q = VLLM(
    model="TheBloke/Llama-2-7b-Chat-AWQ",
    trust_remote_code=True,
    max_new_tokens=512,
    vllm_kwargs={"quantization": "awq"},
)
```

## OpenAI兼容服务器

vLLM可以作为一个服务器进行部署，模仿OpenAI API协议。这使得vLLM可以作为使用OpenAI API的应用程序的替代品。

该服务器可以使用与OpenAI API相同的格式进行查询。

### OpenAI兼容的完成


```python
from langchain_community.llms import VLLMOpenAI

llm = VLLMOpenAI(
    openai_api_key="EMPTY",
    openai_api_base="http://localhost:8000/v1",
    model_name="tiiuae/falcon-7b",
    model_kwargs={"stop": ["."]},
)
print(llm.invoke("Rome is"))
```
```output
 a city that is filled with history, ancient buildings, and art around every corner
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)