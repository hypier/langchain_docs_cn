---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/providers/exa_search.ipynb
---

# Exa 搜索

Exa 的搜索集成存在于其自己的 [合作伙伴包](https://pypi.org/project/langchain-exa/)。您可以通过以下命令安装它：


```python
%pip install -qU langchain-exa
```

为了使用该包，您还需要将 `EXA_API_KEY` 环境变量设置为您的 Exa API 密钥。

## Retriever

您可以在标准检索管道中使用 [`ExaSearchRetriever`](/docs/integrations/tools/exa_search#using-exasearchretriever)。您可以按如下方式导入它

```python
from langchain_exa import ExaSearchRetriever
```

## 工具

您可以使用 Exa 作为代理工具，具体说明请参见 [Exa 工具调用文档](/docs/integrations/tools/exa_search#using-the-exa-sdk-as-langchain-agent-tools)。