---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/aphrodite.ipynb
---

# 阿佛洛狄特引擎

[Aphrodite](https://github.com/PygmalionAI/aphrodite-engine) 是一个开源的大规模推理引擎，旨在为 [PygmalionAI](https://pygmalion.chat) 网站上的数千名用户提供服务。

* 由 vLLM 提供的注意力机制，实现快速吞吐量和低延迟
* 支持多种 SOTA 采样方法
* Exllamav2 GPTQ 内核，在较小批量下实现更好的吞吐量

本笔记本介绍如何将 LLM 与 langchain 和阿佛洛狄特结合使用。

要使用此功能，您需要安装 `aphrodite-engine` Python 包。

```python
##安装使用集成所需的 langchain 包
%pip install -qU langchain-community
```

```python
%pip install --upgrade --quiet  aphrodite-engine==0.4.2
# %pip list | grep aphrodite
```

```python
from langchain_community.llms import Aphrodite

llm = Aphrodite(
    model="PygmalionAI/pygmalion-2-7b",
    trust_remote_code=True,  # hf 模型的必需项
    max_tokens=128,
    temperature=1.2,
    min_p=0.05,
    mirostat_mode=0,  # 更改为 2 使用 mirostat
    mirostat_tau=5.0,
    mirostat_eta=0.1,
)

print(
    llm.invoke(
        '<|system|>进入 RP 模式。你是 Ayumu "Osaka" Kasuga。<|user|>嘿，大阪。告诉我关于你自己的事。<|model|>'
    )
)
```
```output
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] 正在用以下配置初始化阿佛洛狄特引擎：
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] 模型 = 'PygmalionAI/pygmalion-2-7b'
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] 分词器 = 'PygmalionAI/pygmalion-2-7b'
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] tokenizer_mode = auto
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] revision = None
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] trust_remote_code = True
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] DataType = torch.bfloat16
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] 下载目录 = None
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] 模型加载格式 = auto
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] GPU 数量 = 1
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] 量化格式 = None
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] 采样器种子 = 0
[32mINFO 12-15 11:52:48 aphrodite_engine.py:73] 上下文长度 = 4096[0m
[32mINFO 12-15 11:54:07 aphrodite_engine.py:206] # GPU 块: 3826, # CPU 块: 512[0m
``````output
处理的提示: 100%|██████████| 1/1 [00:02<00:00,  2.91s/it]
``````output
我是 Ayumu "Osaka" Kasuga，我是一个狂热的动漫和漫画爱好者！我比较内向，但我一直喜欢阅读书籍、观看动漫和漫画，以及学习日本文化。我最喜欢的动漫系列是《我的英雄学院》、《进击的巨人》和《刀剑神域》。我也非常喜欢阅读漫画系列《海贼王》、《火影忍者》和《银魂》系列。
``````output

```

## 在 LLMChain 中集成模型


```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate

template = """Question: {question}

Answer: Let's think step by step."""
prompt = PromptTemplate.from_template(template)

llm_chain = LLMChain(prompt=prompt, llm=llm)

question = "Who was the US president in the year the first Pokemon game was released?"

print(llm_chain.run(question))
```
```output
Processed prompts: 100%|██████████| 1/1 [00:03<00:00,  3.56s/it]
``````output
 The first Pokemon game was released in Japan on 27 February 1996 (their release dates differ from ours) and it is known as Red and Green. President Bill Clinton was in the White House in the years of 1993, 1994, 1995 and 1996 so this fits.

Answer: Let's think step by step.

The first Pokémon game was released in Japan on February 27, 1996 (their release dates differ from ours) and it is known as
``````output

```

## 分布式推理

Aphrodite 支持分布式张量并行推理和服务。

要使用 LLM 类运行多 GPU 推理，请将 `tensor_parallel_size` 参数设置为您要使用的 GPU 数量。例如，要在 4 个 GPU 上运行推理：

```python
from langchain_community.llms import Aphrodite

llm = Aphrodite(
    model="PygmalionAI/mythalion-13b",
    tensor_parallel_size=4,
    trust_remote_code=True,  # mandatory for hf models
)

llm("What is the future of AI?")
```
```output
2023-12-15 11:41:27,790	INFO worker.py:1636 -- Started a local Ray instance.
``````output
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] Initializing the Aphrodite Engine with the following config:
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] Model = 'PygmalionAI/mythalion-13b'
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] Tokenizer = 'PygmalionAI/mythalion-13b'
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] tokenizer_mode = auto
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] revision = None
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] trust_remote_code = True
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] DataType = torch.float16
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] Download Directory = None
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] Model Load Format = auto
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] Number of GPUs = 4
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] Quantization Format = None
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] Sampler Seed = 0
[32mINFO 12-15 11:41:35 aphrodite_engine.py:73] Context Length = 4096[0m
[32mINFO 12-15 11:43:58 aphrodite_engine.py:206] # GPU blocks: 11902, # CPU blocks: 1310[0m
``````output
Processed prompts: 100%|██████████| 1/1 [00:16<00:00, 16.09s/it]
```


```output
"\n2 years ago StockBot101\nAI is becoming increasingly real and more and more powerful with every year. But what does the future hold for artificial intelligence?\nThere are many possibilities for how AI could evolve and change our world. Some believe that AI will become so advanced that it will take over human jobs, while others believe that AI will be used to augment and assist human workers. There is also the possibility that AI could develop its own consciousness and become self-aware.\nWhatever the future holds, it is clear that AI will continue to play an important role in our lives. Technologies such as machine learning and natural language processing are already transforming industries like healthcare, manufacturing, and transportation. And as AI continues to develop, we can expect even more disruption and innovation across all sectors of the economy.\nSo what exactly are we looking at? What's the future of AI?\nIn the next few years, we can expect AI to be used more and more in healthcare. With the power of machine learning, artificial intelligence can help doctors diagnose diseases earlier and more accurately. It can also be used to develop new treatments and personalize care plans for individual patients.\nManufacturing is another area where AI is already having a big impact. Companies are using robotics and automation to build products faster and with fewer errors. And as AI continues to advance, we can expect even more changes in manufacturing, such as the development of self-driving factories.\nTransportation is another industry that is being transformed by artificial intelligence. Self-driving cars are already being tested on public roads, and it's likely that they will become commonplace in the next decade or so. AI-powered drones are also being developed for use in delivery and even firefighting.\nFinally, artificial intelligence is also poised to have a big impact on customer service and sales. Chatbots and virtual assistants will become more sophisticated, making it easier for businesses to communicate with customers and sell their products.\nThis is just the beginning for artificial intelligence. As the technology continues to develop, we can expect even more amazing advances and innovations. The future of AI is truly limitless.\nWhat do you think the future of AI holds? Do you see any other major"
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)