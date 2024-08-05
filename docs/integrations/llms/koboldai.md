---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/koboldai.ipynb
---

# KoboldAI API

[KoboldAI](https://github.com/KoboldAI/KoboldAI-Client) 是一个“基于浏览器的前端，用于与多个本地和远程AI模型进行AI辅助写作...”。它具有一个公共和本地API，可以在langchain中使用。

本示例介绍了如何使用LangChain与该API。

文档可以在浏览器中通过在端点后添加/api找到（即 http://127.0.0.1/:5000/api）。

```python
from langchain_community.llms import KoboldApiLLM
```

将下面看到的端点替换为在使用 --api 或 --public-api 启动webui后显示的端点。

可选地，您可以传递像 temperature 或 max_length 的参数。

```python
llm = KoboldApiLLM(endpoint="http://192.168.1.144:5000", max_length=80)
```

```python
response = llm.invoke(
    "### Instruction:\nWhat is the first book of the bible?\n### Response:"
)
```

## 相关

- LLM [概念指南](/docs/concepts/#llms)
- LLM [操作指南](/docs/how_to/#llms)