---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/azure_ml.ipynb
---

# Azure ML

[Azure ML](https://azure.microsoft.com/en-us/products/machine-learning/) 是一个用于构建、训练和部署机器学习模型的平台。用户可以在模型目录中探索可部署的模型类型，该目录提供来自不同供应商的基础和通用模型。

本笔记本介绍了如何使用托管在 `Azure ML Online Endpoint` 上的 LLM。


```python
##Installing the langchain packages needed to use the integration
%pip install -qU langchain-community
```


```python
from langchain_community.llms.azureml_endpoint import AzureMLOnlineEndpoint
```

## 设置

您必须 [在 Azure ML 上部署模型](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-use-foundation-models?view=azureml-api-2#deploying-foundation-models-to-endpoints-for-inferencing) 或 [在 Azure AI Studio 上](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/deploy-models-open) 并获取以下参数：

* `endpoint_url`: 由端点提供的 REST 端点 URL。
* `endpoint_api_type`: 在将模型部署到 **专用端点**（托管的基础设施）时使用 `endpoint_type='dedicated'`。在使用 **按需付费** 选项（模型即服务）部署模型时使用 `endpoint_type='serverless'`。
* `endpoint_api_key`: 由端点提供的 API 密钥。
* `deployment_name`: （可选）使用端点的模型的部署名称。

## 内容格式化器

`content_formatter` 参数是一个处理类，用于将 AzureML 端点的请求和响应转换为所需的模式。由于模型目录中有多种模型，每个模型可能以不同的方式处理数据，因此提供了 `ContentFormatterBase` 类，以允许用户根据自己的喜好转换数据。提供了以下内容格式化器：

* `GPT2ContentFormatter`：为 GPT2 格式化请求和响应数据
* `DollyContentFormatter`：为 Dolly-v2 格式化请求和响应数据
* `HFContentFormatter`：为文本生成 Hugging Face 模型格式化请求和响应数据
* `CustomOpenAIContentFormatter`：为遵循 OpenAI API 兼容方案的模型，如 LLaMa2，格式化请求和响应数据。

*注意：`OSSContentFormatter` 正在被弃用，并被 `GPT2ContentFormatter` 替代。逻辑是相同的，但 `GPT2ContentFormatter` 是一个更合适的名称。您仍然可以继续使用 `OSSContentFormatter`，因为这些更改是向后兼容的。*

## 示例

### 示例：LlaMa 2 完成与实时端点


```python
from langchain_community.llms.azureml_endpoint import (
    AzureMLEndpointApiType,
    CustomOpenAIContentFormatter,
)
from langchain_core.messages import HumanMessage

llm = AzureMLOnlineEndpoint(
    endpoint_url="https://<your-endpoint>.<your_region>.inference.ml.azure.com/score",
    endpoint_api_type=AzureMLEndpointApiType.dedicated,
    endpoint_api_key="my-api-key",
    content_formatter=CustomOpenAIContentFormatter(),
    model_kwargs={"temperature": 0.8, "max_new_tokens": 400},
)
response = llm.invoke("Write me a song about sparkling water:")
response
```

在调用时也可以指示模型参数：


```python
response = llm.invoke("Write me a song about sparkling water:", temperature=0.5)
response
```

### 示例：按需付费部署的聊天完成（作为服务的模型）

```python
from langchain_community.llms.azureml_endpoint import (
    AzureMLEndpointApiType,
    CustomOpenAIContentFormatter,
)
from langchain_core.messages import HumanMessage

llm = AzureMLOnlineEndpoint(
    endpoint_url="https://<your-endpoint>.<your_region>.inference.ml.azure.com/v1/completions",
    endpoint_api_type=AzureMLEndpointApiType.serverless,
    endpoint_api_key="my-api-key",
    content_formatter=CustomOpenAIContentFormatter(),
    model_kwargs={"temperature": 0.8, "max_new_tokens": 400},
)
response = llm.invoke("Write me a song about sparkling water:")
response
```

### 示例：自定义内容格式化器

以下是使用 Hugging Face 的摘要模型的示例。

```python
import json
import os
from typing import Dict

from langchain_community.llms.azureml_endpoint import (
    AzureMLOnlineEndpoint,
    ContentFormatterBase,
)


class CustomFormatter(ContentFormatterBase):
    content_type = "application/json"
    accepts = "application/json"

    def format_request_payload(self, prompt: str, model_kwargs: Dict) -> bytes:
        input_str = json.dumps(
            {
                "inputs": [prompt],
                "parameters": model_kwargs,
                "options": {"use_cache": False, "wait_for_model": True},
            }
        )
        return str.encode(input_str)

    def format_response_payload(self, output: bytes) -> str:
        response_json = json.loads(output)
        return response_json[0]["summary_text"]


content_formatter = CustomFormatter()

llm = AzureMLOnlineEndpoint(
    endpoint_api_type="dedicated",
    endpoint_api_key=os.getenv("BART_ENDPOINT_API_KEY"),
    endpoint_url=os.getenv("BART_ENDPOINT_URL"),
    model_kwargs={"temperature": 0.8, "max_new_tokens": 400},
    content_formatter=content_formatter,
)
large_text = """2020年1月7日，Blockberry Creative宣布HaSeul因心理健康问题将不参加Loona的新专辑推广。她被诊断为“间歇性焦虑症状”，并将花时间专注于自己的健康。[39] 2020年2月5日，Loona发布了他们的第二张EP，标题为[#]（读作哈希），以及主打曲“So What”。[40] 尽管HaSeul没有出现在主打曲中，但她的声音出现在专辑的另外三首歌曲中，包括“365”。该EP曾在每日Gaon零售专辑排行榜上登顶第1位，[41] 然后在每周Gaon专辑排行榜上首发第2位。2020年3月12日，Loona凭借“So What”在Mnet的M Countdown上赢得了他们的第一座音乐节目奖杯。[42]

2020年10月19日，Loona发布了他们的第三张EP，标题为[12:00]（读作午夜），[43] 附带第一支单曲“Why Not?”。HaSeul再次没有参与该专辑，出于她自己专注于健康恢复的决定。[44] 该EP成为他们第一张进入Billboard 200的专辑，首发第112位。[45] 11月18日，Loona发布了“Star”的音乐视频，这是[12:00]中的另一首歌曲。[46] “Star”最高排名第40，是Loona首次进入Billboard Mainstream Top 40，使他们成为第二个进入该榜单的K-pop女子团体。[47]

2021年6月1日，Loona宣布他们将在6月28日回归，推出第四张EP，[&]（读作和）。[48] 次日，6月2日，Loona的官方社交媒体账号发布了一则预告，显示十二双眼睛，确认了自2020年初以来一直在休假的成员HaSeul的回归。[49] 6月12日，组合成员YeoJin、Kim Lip、Choerry和Go Won与Cocomong合作发布了歌曲“Yum-Yum”。[50] 9月8日，他们发布了另一首合作歌曲“Yummy-Yummy”。[51] 2021年6月27日，Loona在他们的特别片段结束时宣布，他们将在9月15日以Universal Music Japan子标签EMI Records的名义首次亮相日本。[52] 8月27日，宣布Loona将在9月15日发布双A面单曲“Hula Hoop / Star Seed”，并于10月20日发行实体CD。[53] 12月，Chuu向法院申请暂停与Blockberry Creative的独家合同。[54][55]
"""
summarized_text = llm.invoke(large_text)
print(summarized_text)
```

### 示例：使用 LLMChain 的 Dolly

```python
from langchain.chains import LLMChain
from langchain_community.llms.azureml_endpoint import DollyContentFormatter
from langchain_core.prompts import PromptTemplate

formatter_template = "写一篇关于 {topic} 的 {word_count} 字的文章。"

prompt = PromptTemplate(
    input_variables=["word_count", "topic"], template=formatter_template
)

content_formatter = DollyContentFormatter()

llm = AzureMLOnlineEndpoint(
    endpoint_api_key=os.getenv("DOLLY_ENDPOINT_API_KEY"),
    endpoint_url=os.getenv("DOLLY_ENDPOINT_URL"),
    model_kwargs={"temperature": 0.8, "max_tokens": 300},
    content_formatter=content_formatter,
)

chain = LLMChain(llm=llm, prompt=prompt)
print(chain.invoke({"word_count": 100, "topic": "如何交朋友"}))
```

## 序列化 LLM
您还可以保存和加载 LLM 配置


```python
from langchain_community.llms.loading import load_llm

save_llm = AzureMLOnlineEndpoint(
    deployment_name="databricks-dolly-v2-12b-4",
    model_kwargs={
        "temperature": 0.2,
        "max_tokens": 150,
        "top_p": 0.8,
        "frequency_penalty": 0.32,
        "presence_penalty": 72e-3,
    },
)
save_llm.save("azureml.json")
loaded_llm = load_llm("azureml.json")

print(loaded_llm)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)