---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/sagemaker.ipynb
---

# SageMakerEndpoint

[Amazon SageMaker](https://aws.amazon.com/sagemaker/) 是一个可以构建、训练和部署机器学习（ML）模型的系统，适用于任何用例，提供完全托管的基础设施、工具和工作流程。

本笔记本介绍了如何使用托管在 `SageMaker endpoint` 上的 LLM。

```python
!pip3 install langchain boto3
```

## 设置

您需要设置以下 `SagemakerEndpoint` 调用的必要参数：
- `endpoint_name`: 已部署的 Sagemaker 模型的端点名称。
    必须在 AWS 区域内唯一。
- `credentials_profile_name`: ~/.aws/credentials 或 ~/.aws/config 文件中配置文件的名称，该文件中指定了访问密钥或角色信息。
    如果未指定，将使用默认凭证配置文件，或者如果在 EC2 实例上，将使用 IMDS 中的凭证。
    参见：https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html

## 示例


```python
from langchain_core.documents import Document
```


```python
example_doc_1 = """
Peter 和 Elizabeth 乘出租车前往市区参加晚会。在派对上，Elizabeth 突然晕倒，被紧急送往医院。
由于她被诊断出脑部受伤，医生告诉 Peter 要陪在她身边，直到她康复。
因此，Peter 在医院陪伴她 3 天，没有离开。
"""

docs = [
    Document(
        page_content=example_doc_1,
    )
]
```

## 使用外部 boto3 会话进行初始化的示例

### 跨账户场景

```python
import json
from typing import Dict

import boto3
from langchain.chains.question_answering import load_qa_chain
from langchain_community.llms import SagemakerEndpoint
from langchain_community.llms.sagemaker_endpoint import LLMContentHandler
from langchain_core.prompts import PromptTemplate

query = """Elizabeth住院多久？
"""

prompt_template = """使用以下上下文来回答最后的问题。

{context}

问题: {question}
答案:"""
PROMPT = PromptTemplate(
    template=prompt_template, input_variables=["context", "question"]
)

roleARN = "arn:aws:iam::123456789:role/cross-account-role"
sts_client = boto3.client("sts")
response = sts_client.assume_role(
    RoleArn=roleARN, RoleSessionName="CrossAccountSession"
)

client = boto3.client(
    "sagemaker-runtime",
    region_name="us-west-2",
    aws_access_key_id=response["Credentials"]["AccessKeyId"],
    aws_secret_access_key=response["Credentials"]["SecretAccessKey"],
    aws_session_token=response["Credentials"]["SessionToken"],
)


class ContentHandler(LLMContentHandler):
    content_type = "application/json"
    accepts = "application/json"

    def transform_input(self, prompt: str, model_kwargs: Dict) -> bytes:
        input_str = json.dumps({"inputs": prompt, "parameters": model_kwargs})
        return input_str.encode("utf-8")

    def transform_output(self, output: bytes) -> str:
        response_json = json.loads(output.read().decode("utf-8"))
        return response_json[0]["generated_text"]


content_handler = ContentHandler()

chain = load_qa_chain(
    llm=SagemakerEndpoint(
        endpoint_name="endpoint-name",
        client=client,
        model_kwargs={"temperature": 1e-10},
        content_handler=content_handler,
    ),
    prompt=PROMPT,
)

chain({"input_documents": docs, "question": query}, return_only_outputs=True)
```


```python
import json
from typing import Dict

from langchain.chains.question_answering import load_qa_chain
from langchain_community.llms import SagemakerEndpoint
from langchain_community.llms.sagemaker_endpoint import LLMContentHandler
from langchain_core.prompts import PromptTemplate

query = """Elizabeth住院多久？
"""

prompt_template = """使用以下上下文来回答最后的问题。

{context}

问题: {question}
答案:"""
PROMPT = PromptTemplate(
    template=prompt_template, input_variables=["context", "question"]
)


class ContentHandler(LLMContentHandler):
    content_type = "application/json"
    accepts = "application/json"

    def transform_input(self, prompt: str, model_kwargs: Dict) -> bytes:
        input_str = json.dumps({"inputs": prompt, "parameters": model_kwargs})
        return input_str.encode("utf-8")

    def transform_output(self, output: bytes) -> str:
        response_json = json.loads(output.read().decode("utf-8"))
        return response_json[0]["generated_text"]


content_handler = ContentHandler()

chain = load_qa_chain(
    llm=SagemakerEndpoint(
        endpoint_name="endpoint-name",
        credentials_profile_name="credentials-profile-name",
        region_name="us-west-2",
        model_kwargs={"temperature": 1e-10},
        content_handler=content_handler,
    ),
    prompt=PROMPT,
)

chain({"input_documents": docs, "question": query}, return_only_outputs=True)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)