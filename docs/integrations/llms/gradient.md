---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/gradient.ipynb
---

# 渐变

`Gradient` 允许通过简单的 web API 对 LLM 进行微调和获取补全。

本笔记本介绍如何使用 Langchain 与 [Gradient](https://gradient.ai/)。

## 导入


```python
from langchain.chains import LLMChain
from langchain_community.llms import GradientLLM
from langchain_core.prompts import PromptTemplate
```

## 设置环境 API 密钥
确保从 Gradient AI 获取您的 API 密钥。您将获得 $10 的免费积分以测试和微调不同的模型。

```python
import os
from getpass import getpass

if not os.environ.get("GRADIENT_ACCESS_TOKEN", None):
    # 访问令牌在 https://auth.gradient.ai/select-workspace
    os.environ["GRADIENT_ACCESS_TOKEN"] = getpass("gradient.ai 访问令牌:")
if not os.environ.get("GRADIENT_WORKSPACE_ID", None):
    # 在 `$ gradient workspace list` 中列出的 `ID`
    # 登录后也在 https://auth.gradient.ai/select-workspace 显示
    os.environ["GRADIENT_WORKSPACE_ID"] = getpass("gradient.ai 工作区 ID:")
```

可选：验证您的环境变量 ```GRADIENT_ACCESS_TOKEN``` 和 ```GRADIENT_WORKSPACE_ID``` 以获取当前部署的模型。使用 `gradientai` Python 包。

```python
%pip install --upgrade --quiet  gradientai
```
```output
Requirement already satisfied: gradientai in /home/michi/.venv/lib/python3.10/site-packages (1.0.0)
Requirement already satisfied: aenum>=3.1.11 in /home/michi/.venv/lib/python3.10/site-packages (from gradientai) (3.1.15)
Requirement already satisfied: pydantic<2.0.0,>=1.10.5 in /home/michi/.venv/lib/python3.10/site-packages (from gradientai) (1.10.12)
Requirement already satisfied: python-dateutil>=2.8.2 in /home/michi/.venv/lib/python3.10/site-packages (from gradientai) (2.8.2)
Requirement already satisfied: urllib3>=1.25.3 in /home/michi/.venv/lib/python3.10/site-packages (from gradientai) (1.26.16)
Requirement already satisfied: typing-extensions>=4.2.0 in /home/michi/.venv/lib/python3.10/site-packages (from pydantic<2.0.0,>=1.10.5->gradientai) (4.5.0)
Requirement already satisfied: six>=1.5 in /home/michi/.venv/lib/python3.10/site-packages (from python-dateutil>=2.8.2->gradientai) (1.16.0)
```

```python
import gradientai

client = gradientai.Gradient()

models = client.list_models(only_base=True)
for model in models:
    print(model.id)
```
```output
99148c6d-c2a0-4fbe-a4a7-e7c05bdb8a09_base_ml_model
f0b97d96-51a8-4040-8b22-7940ee1fa24e_base_ml_model
cc2dafce-9e6e-4a23-a918-cad6ba89e42e_base_ml_model
```

```python
new_model = models[-1].create_model_adapter(name="my_model_adapter")
new_model.id, new_model.name
```

```output
('674119b5-f19e-4856-add2-767ae7f7d7ef_model_adapter', 'my_model_adapter')
```

## 创建 Gradient 实例
您可以指定不同的参数，例如模型、生成的最大令牌数、温度等。

由于我们稍后想要微调我们的模型，因此我们选择具有 ID `674119b5-f19e-4856-add2-767ae7f7d7ef_model_adapter` 的 model_adapter，但您可以使用任何基础或可微调的模型。

```python
llm = GradientLLM(
    # `ID` listed in `$ gradient model list`
    model="674119b5-f19e-4856-add2-767ae7f7d7ef_model_adapter",
    # # optional: set new credentials, they default to environment variables
    # gradient_workspace_id=os.environ["GRADIENT_WORKSPACE_ID"],
    # gradient_access_token=os.environ["GRADIENT_ACCESS_TOKEN"],
    model_kwargs=dict(max_generated_token_count=128),
)
```

## 创建一个提示模板
我们将为问答创建一个提示模板。

```python
template = """Question: {question}

Answer: """

prompt = PromptTemplate.from_template(template)
```

## 初始化 LLMChain


```python
llm_chain = LLMChain(prompt=prompt, llm=llm)
```

## 运行 LLMChain
提供一个问题并运行 LLMChain。

```python
question = "What NFL team won the Super Bowl in 1994?"

llm_chain.run(question=question)
```

```output
'\nThe San Francisco 49ers won the Super Bowl in 1994.'
```

# 通过微调提高结果（可选）
好吧 - 这是错误的 - 旧金山49人队并没有获胜。
正确的答案是`达拉斯牛仔队！`。

让我们通过使用PromptTemplate对正确答案进行微调来提高正确答案的概率。


```python
dataset = [
    {
        "inputs": template.format(question="1994年哪支NFL球队赢得了超级碗？")
        + " 达拉斯牛仔队！"
    }
]
dataset
```



```output
[{'inputs': '问题：1994年哪支NFL球队赢得了超级碗？\n\n答案：  达拉斯牛仔队！'}]
```



```python
new_model.fine_tune(samples=dataset)
```



```output
FineTuneResponse(number_of_trainable_tokens=27, sum_loss=78.17996)
```



```python
# 我们可以保持llm_chain，因为注册的模型刚刚在gradient.ai服务器上刷新。
llm_chain.run(question=question)
```



```output
'达拉斯牛仔队'
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)