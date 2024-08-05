---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/octoai.ipynb
---

# OctoAI

[OctoAI](https://docs.octoai.cloud/docs) 提供了便捷的高效计算访问，并使用户能够将所选的 AI 模型集成到应用程序中。`OctoAI` 计算服务帮助您轻松运行、调整和扩展 AI 应用程序。

此示例介绍了如何使用 LangChain 与 `OctoAI` [LLM 端点](https://octoai.cloud/templates) 进行交互。

## 设置

要运行我们的示例应用程序，您需要执行两个简单步骤：

1. 从 [您的 OctoAI 账户页面](https://octoai.cloud/settings) 获取 API 令牌。
   
2. 将您的 API 密钥粘贴到下面的代码单元中。

注意：如果您想使用不同的 LLM 模型，您可以将模型容器化，并按照 [从 Python 构建容器](https://octo.ai/docs/bring-your-own-model/advanced-build-a-container-from-scratch-in-python) 和 [从容器创建自定义端点](https://octo.ai/docs/bring-your-own-model/create-custom-endpoints-from-a-container/create-custom-endpoints-from-a-container) 的说明自己创建一个自定义 OctoAI 端点，然后更新您的 `OCTOAI_API_BASE` 环境变量。



```python
import os

os.environ["OCTOAI_API_TOKEN"] = "OCTOAI_API_TOKEN"
```


```python
from langchain.chains import LLMChain
from langchain_community.llms.octoai_endpoint import OctoAIEndpoint
from langchain_core.prompts import PromptTemplate
```

## 示例


```python
template = """以下是描述任务的指令。写一个适当完成请求的回应。\n 指令:\n{question}\n 回应: """
prompt = PromptTemplate.from_template(template)
```


```python
llm = OctoAIEndpoint(
    model_name="llama-2-13b-chat-fp16",
    max_tokens=200,
    presence_penalty=0,
    temperature=0.1,
    top_p=0.9,
)
```


```python
question = "谁是列奥纳多·达·芬奇？"

chain = prompt | llm

print(chain.invoke(question))
```

列奥纳多·达·芬奇是真正的文艺复兴人。他于1452年出生在意大利的文奇，以在艺术、科学、工程和数学等多个领域的工作而闻名。他被认为是有史以来最伟大的画家之一，他最著名的作品包括《蒙娜丽莎》和《最后的晚餐》。除了他的艺术，达·芬奇在工程和解剖学方面也做出了重要贡献，他的机器和发明设计在几个世纪前就超出了他的时代。他还因其广泛的日记和绘图而闻名，这些提供了他思想和理念的宝贵见解。达·芬奇的遗产继续激励和影响着世界各地的艺术家、科学家和思想家。

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)