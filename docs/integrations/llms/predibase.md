---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/predibase.ipynb
---

# Predibase

[Predibase](https://predibase.com/) 允许您训练、微调和部署任何机器学习模型——从线性回归到大型语言模型。

本示例演示了如何使用 Langchain 与部署在 Predibase 上的模型。

# 设置

要运行此笔记本，您需要一个 [Predibase 账户](https://predibase.com/free-trial/?utm_source=langchain) 和一个 [API 密钥](https://docs.predibase.com/sdk-guide/intro)。

您还需要安装 Predibase Python 包：

```python
%pip install --upgrade --quiet  predibase
import os

os.environ["PREDIBASE_API_TOKEN"] = "{PREDIBASE_API_TOKEN}"
```

## 初始调用


```python
from langchain_community.llms import Predibase

model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
)
```


```python
from langchain_community.llms import Predibase

# 使用托管在Predibase上的微调适配器（必须指定adapter_version）。
model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # 可选参数（如果省略，则默认为最新的Predibase SDK版本）
    adapter_id="e2e_nlg",
    adapter_version=1,
)
```


```python
from langchain_community.llms import Predibase

# 使用托管在HuggingFace上的微调适配器（adapter_version不适用，将被忽略）。
model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # 可选参数（如果省略，则默认为最新的Predibase SDK版本）
    adapter_id="predibase/e2e_nlg",
)
```


```python
response = model.invoke("Can you recommend me a nice dry wine?")
print(response)
```

## 链式调用设置


```python
from langchain_community.llms import Predibase

model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # optional parameter (defaults to the latest Predibase SDK version if omitted)
)
```


```python
# 使用托管在Predibase的微调适配器（必须指定adapter_version）。
model = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # optional parameter (defaults to the latest Predibase SDK version if omitted)
    adapter_id="e2e_nlg",
    adapter_version=1,
)
```


```python
# 使用托管在HuggingFace的微调适配器（adapter_version不适用，将被忽略）。
llm = Predibase(
    model="mistral-7b",
    predibase_api_key=os.environ.get("PREDIBASE_API_TOKEN"),
    predibase_sdk_version=None,  # optional parameter (defaults to the latest Predibase SDK version if omitted)
    adapter_id="predibase/e2e_nlg",
)
```

##  SequentialChain


```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate
```


```python
# 这是一个 LLMChain，根据剧本标题撰写概要。
template = """You are a playwright. Given the title of play, it is your job to write a synopsis for that title.

Title: {title}
Playwright: This is a synopsis for the above play:"""
prompt_template = PromptTemplate(input_variables=["title"], template=template)
synopsis_chain = LLMChain(llm=llm, prompt=prompt_template)
```


```python
# 这是一个 LLMChain，根据概要撰写剧评。
template = """You are a play critic from the New York Times. Given the synopsis of play, it is your job to write a review for that play.

Play Synopsis:
{synopsis}
Review from a New York Times play critic of the above play:"""
prompt_template = PromptTemplate(input_variables=["synopsis"], template=template)
review_chain = LLMChain(llm=llm, prompt=prompt_template)
```


```python
# 这是整体链，我们按顺序运行这两个链。
from langchain.chains import SimpleSequentialChain

overall_chain = SimpleSequentialChain(
    chains=[synopsis_chain, review_chain], verbose=True
)
```


```python
review = overall_chain.run("Tragedy at sunset on the beach")
```

## 微调的 LLM（使用您自己从 Predibase 微调的 LLM）


```python
from langchain_community.llms import Predibase

model = Predibase(
    model="my-base-LLM",
    predibase_api_key=os.environ.get(
        "PREDIBASE_API_TOKEN"
    ),  # Adapter argument is optional.
    predibase_sdk_version=None,  # optional parameter (defaults to the latest Predibase SDK version if omitted)
    adapter_id="my-finetuned-adapter-id",  # Supports both, Predibase-hosted and HuggingFace-hosted adapter repositories.
    adapter_version=1,  # required for Predibase-hosted adapters (ignored for HuggingFace-hosted adapters)
)
# replace my-base-LLM with the name of your choice of a serverless base model in Predibase
```


```python
# response = model.invoke("Can you help categorize the following emails into positive, negative, and neutral?")
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)