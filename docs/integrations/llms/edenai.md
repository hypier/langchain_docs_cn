---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/edenai.ipynb
---

# Eden AI

Eden AI 正在通过联合最佳的 AI 提供商，彻底改变 AI 领域，赋能用户释放无限可能，挖掘人工智能的真正潜力。通过一个全面且无忧的平台，它使用户能够快速将 AI 功能部署到生产中，通过单一 API 轻松访问全面的 AI 能力。 (网站: https://edenai.co/)

这个示例介绍了如何使用 LangChain 与 Eden AI 模型进行交互

-----------------------------------------------------------------------------------

访问 EDENAI 的 API 需要一个 API 密钥，

您可以通过创建一个帐户 https://app.edenai.run/user/register 并前往此处 https://app.edenai.run/admin/account/settings 获取密钥。

一旦我们有了密钥，我们想通过运行以下命令将其设置为环境变量：

```bash
export EDENAI_API_KEY="..."
```

如果您不想设置环境变量，可以在初始化 EdenAI LLM 类时直接通过名为 edenai_api_key 的参数传递密钥：

```python
from langchain_community.llms import EdenAI
```

```python
llm = EdenAI(edenai_api_key="...", provider="openai", temperature=0.2, max_tokens=250)
```

## 调用模型


EdenAI API 汇集了多个提供商，每个提供商都提供多个模型。

要访问特定模型，您只需在实例化时添加 'model'。

例如，让我们探索 OpenAI 提供的模型，如 GPT3.5

### 文本生成


```python
from langchain.chains import LLMChain
from langchain_core.prompts import PromptTemplate

llm = EdenAI(
    feature="text",
    provider="openai",
    model="gpt-3.5-turbo-instruct",
    temperature=0.2,
    max_tokens=250,
)

prompt = """
User: Answer the following yes/no question by reasoning step by step. Can a dog drive a car?
Assistant:
"""

llm(prompt)
```

### 图像生成


```python
import base64
from io import BytesIO

from PIL import Image


def print_base64_image(base64_string):
    # Decode the base64 string into binary data
    decoded_data = base64.b64decode(base64_string)

    # Create an in-memory stream to read the binary data
    image_stream = BytesIO(decoded_data)

    # Open the image using PIL
    image = Image.open(image_stream)

    # Display the image
    image.show()
```


```python
text2image = EdenAI(feature="image", provider="openai", resolution="512x512")
```


```python
image_output = text2image("一只骑摩托车的猫，毕加索风格")
```


```python
print_base64_image(image_output)
```

### 使用回调的文本生成


```python
from langchain_community.llms import EdenAI
from langchain_core.callbacks import StreamingStdOutCallbackHandler

llm = EdenAI(
    callbacks=[StreamingStdOutCallbackHandler()],
    feature="text",
    provider="openai",
    temperature=0.2,
    max_tokens=250,
)
prompt = """
User: Answer the following yes/no question by reasoning step by step. Can a dog drive a car?
Assistant:
"""
print(llm.invoke(prompt))
```

## 链式调用


```python
from langchain.chains import LLMChain, SimpleSequentialChain
from langchain_core.prompts import PromptTemplate
```


```python
llm = EdenAI(feature="text", provider="openai", temperature=0.2, max_tokens=250)
text2image = EdenAI(feature="image", provider="openai", resolution="512x512")
```


```python
prompt = PromptTemplate(
    input_variables=["product"],
    template="为制造 {product} 的公司起个好名字是什么？",
)

chain = LLMChain(llm=llm, prompt=prompt)
```


```python
second_prompt = PromptTemplate(
    input_variables=["company_name"],
    template="为这个公司写一个logo的描述：{company_name}，logo中不应包含任何文字",
)
chain_two = LLMChain(llm=llm, prompt=second_prompt)
```


```python
third_prompt = PromptTemplate(
    input_variables=["company_logo_description"],
    template="{company_logo_description}",
)
chain_three = LLMChain(llm=text2image, prompt=third_prompt)
```


```python
# 运行链，仅指定第一个链的输入变量。
overall_chain = SimpleSequentialChain(
    chains=[chain, chain_two, chain_three], verbose=True
)
output = overall_chain.run("帽子")
```


```python
# 打印图像
print_base64_image(output)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)