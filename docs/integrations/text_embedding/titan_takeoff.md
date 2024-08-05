---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/titan_takeoff.ipynb
---

# Titan Takeoff

`TitanML` 帮助企业通过我们的训练、压缩和推理优化平台构建和部署更好、更小、更便宜和更快的 NLP 模型。

我们的推理服务器 [Titan Takeoff](https://docs.titanml.co/docs/intro) 使您能够通过单个命令在本地硬件上部署 LLM。大多数嵌入模型开箱即用，如果您在使用特定模型时遇到问题，请通过 hello@titanml.co 联系我们。

## 示例用法
以下是一些有助于开始使用 Titan Takeoff Server 的示例。在运行这些命令之前，您需要确保 Takeoff Server 已在后台启动。有关更多信息，请参见 [启动 Takeoff 的文档页面](https://docs.titanml.co/docs/Docs/launching/)。


```python
import time

from langchain_community.embeddings import TitanTakeoffEmbed
```

### 示例 1
基本用法假设 Takeoff 正在您的机器上运行，并使用其默认端口（即 localhost:3000）。

```python
embed = TitanTakeoffEmbed()
output = embed.embed_query(
    "What is the weather in London in August?", consumer_group="embed"
)
print(output)
```

### 示例 2
开始使用 TitanTakeoffEmbed Python Wrapper 的读者。如果您在首次启动 Takeoff 时未创建任何读者，或者您想添加另一个读者，可以在初始化 TitanTakeoffEmbed 对象时进行。只需将您想要启动的模型列表作为 `models` 参数传递即可。

您可以使用 `embed.query_documents` 一次嵌入多个文档。预期输入是一个字符串列表，而不是 `embed_query` 方法所期望的单个字符串。

```python
# Model config for the embedding model, where you can specify the following parameters:
#   model_name (str): The name of the model to use
#   device: (str): The device to use for inference, cuda or cpu
#   consumer_group (str): The consumer group to place the reader into
embedding_model = {
    "model_name": "BAAI/bge-large-en-v1.5",
    "device": "cpu",
    "consumer_group": "embed",
}
embed = TitanTakeoffEmbed(models=[embedding_model])

# The model needs time to spin up, length of time need will depend on the size of model and your network connection speed
time.sleep(60)

prompt = "What is the capital of France?"
# We specified "embed" consumer group so need to send request to the same consumer group so it hits our embedding model and not others
output = embed.embed_query(prompt, consumer_group="embed")
print(output)
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)