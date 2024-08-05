# Marqo

本页面介绍如何在 LangChain 中使用 Marqo 生态系统。

### **什么是 Marqo？**

Marqo 是一个张量搜索引擎，使用存储在内存 HNSW 索引中的嵌入来实现尖端搜索速度。Marqo 可以通过水平索引分片扩展到数亿文档索引，并允许异步和非阻塞的数据上传和搜索。Marqo 使用来自 PyTorch、Huggingface、OpenAI 等的最新机器学习模型。您可以使用预配置的模型或自定义模型。内置的 ONNX 支持和转换允许在 CPU 和 GPU 上实现更快的推理和更高的吞吐量。

由于 Marqo 包含自己的推理，您的文档可以混合文本和图像，您可以将 Marqo 索引与其他系统中的数据一起引入 langchain 生态系统，而不必担心您的嵌入是否兼容。

Marqo 的部署灵活，您可以使用我们的 docker 镜像自行开始，或[联系我们了解我们的托管云服务！](https://www.marqo.ai/pricing)

要使用我们的 docker 镜像在本地运行 Marqo，[请参阅我们的入门指南。](https://docs.marqo.ai/latest/)

## 安装与设置
- 使用 `pip install marqo` 安装 Python SDK

## 包装器

### VectorStore

存在一个围绕 Marqo 索引的封装，允许您在 vectorstore 框架内使用它们。Marqo 让您可以从多种模型中选择以生成嵌入，并暴露一些预处理配置。

Marqo vectorstore 还可以与现有的多模型索引一起工作，您的文档可以混合图像和文本，更多信息请参见 [我们的文档](https://docs.marqo.ai/latest/#multi-modal-and-cross-modal-search)。请注意，使用现有的多模态索引实例化 Marqo vectorstore 将禁用通过 langchain vectorstore `add_texts` 方法添加任何新文档的能力。

要导入此 vectorstore：
```python
from langchain_community.vectorstores import Marqo
```

有关 Marqo 封装及其一些独特功能的更详细操作，请参见 [此笔记本](/docs/integrations/vectorstores/marqo)