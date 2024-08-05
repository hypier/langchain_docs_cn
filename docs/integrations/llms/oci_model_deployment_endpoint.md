---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/oci_model_deployment_endpoint.ipynb
---

# OCI 数据科学模型部署端点

[OCI 数据科学](https://docs.oracle.com/en-us/iaas/data-science/using/home.htm) 是一个完全托管和无服务器的平台，供数据科学团队在 Oracle Cloud Infrastructure 中构建、训练和管理机器学习模型。

本笔记本介绍如何使用托管在 [OCI 数据科学模型部署](https://docs.oracle.com/en-us/iaas/data-science/using/model-dep-about.htm) 上的 LLM。

为了进行身份验证，使用了 [oracle-ads](https://accelerated-data-science.readthedocs.io/en/latest/user_guide/cli/authentication.html) 自动加载调用端点所需的凭据。

```python
!pip3 install oracle-ads
```

## Prerequisites

### 部署模型
查看 [Oracle GitHub 示例库](https://github.com/oracle-samples/oci-data-science-ai-samples/tree/main/model-deployment/containers/llama2) 了解如何在 OCI 数据科学模型部署中部署您的 llm。

### Policy
确保拥有访问 OCI 数据科学模型部署端点所需的 [policy](https://docs.oracle.com/en-us/iaas/data-science/using/model-dep-policies-auth.htm#model_dep_policies_auth__predict-endpoint)。

## Settings

### vLLM
在部署模型后，您必须设置`OCIModelDeploymentVLLM`调用的以下必要参数：

- **`endpoint`**：来自已部署模型的模型HTTP端点，例如`https://<MD_OCID>/predict`。
- **`model`**：模型的位置。

### 文本生成推理 (TGI)
您需要设置 `OCIModelDeploymentTGI` 调用的以下必需参数：

- **`endpoint`**: 从已部署模型获取的模型 HTTP 端点，例如 `https://<MD_OCID>/predict`。

### 认证

您可以通过广告或环境变量设置认证。当您在 OCI 数据科学笔记本会话中工作时，可以利用资源主体访问其他 OCI 资源。请查看 [这里](https://accelerated-data-science.readthedocs.io/en/latest/user_guide/cli/authentication.html) 以了解更多选项。

```
# 代码示例
print("Hello, World!")
```

## 示例


```python
import ads
from langchain_community.llms import OCIModelDeploymentVLLM

# 通过 ads 设置认证
# 使用资源主体在具有资源主体基础的
# 认证配置的 OCI 服务中操作
ads.set_auth("resource_principal")

# 创建 OCI 模型部署端点的实例
# 将端点 URI 和模型名称替换为您自己的
llm = OCIModelDeploymentVLLM(endpoint="https://<MD_OCID>/predict", model="model_name")

# 运行 LLM
llm.invoke("Who is the first president of United States?")
```


```python
import os

from langchain_community.llms import OCIModelDeploymentTGI

# 通过环境变量设置认证
# 当您在本地工作站或不支持
# 资源主体的平台上工作时，请使用 API 密钥设置。
os.environ["OCI_IAM_TYPE"] = "api_key"
os.environ["OCI_CONFIG_PROFILE"] = "default"
os.environ["OCI_CONFIG_LOCATION"] = "~/.oci"

# 通过环境变量设置端点
# 将端点 URI 替换为您自己的
os.environ["OCI_LLM_ENDPOINT"] = "https://<MD_OCID>/predict"

# 创建 OCI 模型部署端点的实例
llm = OCIModelDeploymentTGI()

# 运行 LLM
llm.invoke("Who is the first president of United States?")
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)