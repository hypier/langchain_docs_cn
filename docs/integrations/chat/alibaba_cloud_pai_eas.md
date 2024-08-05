---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/chat/alibaba_cloud_pai_eas.ipynb
sidebar_label: 阿里云 PAI EAS
---

# 阿里云 PAI EAS

>[阿里云 PAI（人工智能平台）](https://www.alibabacloud.com/help/en/pai/?spm=a2c63.p38356.0.0.c26a426ckrxUwZ) 是一个轻量级且高性价比的机器学习平台，采用云原生技术。它为您提供端到端的建模服务。它基于数十亿特征和数百亿样本，在超过 100 种场景中加速模型训练。

>[阿里云机器学习平台（Machine Learning Platform for AI）](https://www.alibabacloud.com/help/en/machine-learning-platform-for-ai/latest/what-is-machine-learning-pai) 是一个面向企业和开发者的机器学习或深度学习工程平台。它提供易于使用、高性价比、高性能且易于扩展的插件，可应用于各种行业场景。凭借超过 140 种内置优化算法，`Machine Learning Platform for AI` 提供全流程的 AI 工程能力，包括数据标注（`PAI-iTAG`）、模型构建（`PAI-Designer` 和 `PAI-DSW`）、模型训练（`PAI-DLC`）、编译优化和推理部署（`PAI-EAS`）。
>
>`PAI-EAS` 支持不同类型的硬件资源，包括 CPU 和 GPU，具有高吞吐量和低延迟。它允许您通过几次点击部署大规模复杂模型，并实时进行弹性缩放和扩展。它还提供全面的运维和监控系统。

## 设置 EAS 服务

设置环境变量以初始化 EAS 服务的 URL 和令牌。有关更多信息，请使用 [此文档](https://www.alibabacloud.com/help/en/pai/user-guide/service-deployment/)。

```bash
export EAS_SERVICE_URL=XXX
export EAS_SERVICE_TOKEN=XXX
```
另一个选项是使用以下代码：

```python
import os

from langchain_community.chat_models import PaiEasChatEndpoint
from langchain_core.language_models.chat_models import HumanMessage

os.environ["EAS_SERVICE_URL"] = "Your_EAS_Service_URL"
os.environ["EAS_SERVICE_TOKEN"] = "Your_EAS_Service_Token"
chat = PaiEasChatEndpoint(
    eas_service_url=os.environ["EAS_SERVICE_URL"],
    eas_service_token=os.environ["EAS_SERVICE_TOKEN"],
)
```

## 运行聊天模型

您可以使用默认设置调用 EAS 服务，如下所示：

```python
output = chat.invoke([HumanMessage(content="write a funny joke")])
print("output:", output)
```

或者，使用新的推理参数调用 EAS 服务：

```python
kwargs = {"temperature": 0.8, "top_p": 0.8, "top_k": 5}
output = chat.invoke([HumanMessage(content="write a funny joke")], **kwargs)
print("output:", output)
```

或者，运行流调用以获取流响应：

```python
outputs = chat.stream([HumanMessage(content="hi")], streaming=True)
for output in outputs:
    print("stream output:", output)
```

## 相关

- 聊天模型 [概念指南](/docs/concepts/#chat-models)
- 聊天模型 [操作指南](/docs/how_to/#chat-models)