---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/nomic.ipynb
---

# Nomic

Nomic 目前提供两个产品：

- Atlas：他们的视觉数据引擎
- GPT4All：他们的开源边缘语言模型生态系统

Nomic 集成存在于其自己的 [合作伙伴包](https://pypi.org/project/langchain-nomic/)。你可以通过以下方式安装它：


```python
%pip install -qU langchain-nomic
```

目前，你可以按如下方式导入他们托管的 [嵌入模型](/docs/integrations/text_embedding/nomic)：


```python
from langchain_nomic import NomicEmbeddings
```