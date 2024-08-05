---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/text_embedding/ipex_llm_gpu.ipynb
---

# 在 Intel GPU 上使用 IPEX-LLM 的本地 BGE 嵌入

> [IPEX-LLM](https://github.com/intel-analytics/ipex-llm) 是一个用于在 Intel CPU 和 GPU（例如，带有 iGPU 的本地 PC、离散 GPU，如 Arc、Flex 和 Max）上以极低延迟运行 LLM 的 PyTorch 库。

本示例介绍了如何使用 LangChain 在 Intel GPU 上通过 `ipex-llm` 优化执行嵌入任务。这对于 RAG、文档问答等应用将非常有帮助。

> **注意**
>
> 建议仅让使用 Intel Arc A 系列 GPU（不包括 Intel Arc A300 系列或 Pro A60）的 Windows 用户直接运行此 Jupyter notebook。对于其他情况（例如，Linux 用户、Intel iGPU 等），建议在终端中使用 Python 脚本运行代码，以获得最佳体验。

## 安装前提条件
为了在Intel GPU上使用IPEX-LLM，有几个工具安装和环境准备的前提步骤。

如果您是Windows用户，请访问[在Windows上安装带Intel GPU的IPEX-LLM指南](https://ipex-llm.readthedocs.io/en/latest/doc/LLM/Quickstart/install_windows_gpu.html)，并按照[安装前提条件](https://ipex-llm.readthedocs.io/en/latest/doc/LLM/Quickstart/install_windows_gpu.html#install-prerequisites)更新GPU驱动程序（可选）并安装Conda。

如果您是Linux用户，请访问[在Linux上安装带Intel GPU的IPEX-LLM](https://ipex-llm.readthedocs.io/en/latest/doc/LLM/Quickstart/install_linux_gpu.html)，并按照[**安装前提条件**](https://ipex-llm.readthedocs.io/en/latest/doc/LLM/Quickstart/install_linux_gpu.html#install-prerequisites)安装GPU驱动程序，Intel® oneAPI基础工具包2024.0和Conda。

## 设置

在安装完先决条件后，您应该已经创建了一个包含所有先决条件的 conda 环境。**在该 conda 环境中启动 jupyter 服务**：

```python
%pip install -qU langchain langchain-community
```

安装 IPEX-LLM 以优化 Intel GPU，以及 `sentence-transformers`。

```python
%pip install --pre --upgrade ipex-llm[xpu] --extra-index-url https://pytorch-extension.intel.com/release-whl/stable/xpu/us/
%pip install sentence-transformers
```

> **注意**
>
> 您也可以使用 `https://pytorch-extension.intel.com/release-whl/stable/xpu/cn/` 作为额外索引 URL。

## 运行时配置

为了获得最佳性能，建议根据您的设备设置几个环境变量：

### 针对使用 Intel Core Ultra 集成 GPU 的 Windows 用户


```python
import os

os.environ["SYCL_CACHE_PERSISTENT"] = "1"
os.environ["BIGDL_LLM_XMX_DISABLED"] = "1"
```

### 对于使用 Intel Arc A 系列 GPU 的 Windows 用户


```python
import os

os.environ["SYCL_CACHE_PERSISTENT"] = "1"
```

> **注意**
>
> 每个模型第一次在 Intel iGPU/Intel Arc A300 系列或 Pro A60 上运行时，可能需要几分钟进行编译。
>
> 对于其他类型的 GPU，请参阅 [这里](https://ipex-llm.readthedocs.io/en/latest/doc/LLM/Overview/install_gpu.html#runtime-configuration) 以获取 Windows 用户的信息，以及 [这里](https://ipex-llm.readthedocs.io/en/latest/doc/LLM/Overview/install_gpu.html#id5) 以获取 Linux 用户的信息。

## 基本用法

在初始化 `IpexLLMBgeEmbeddings` 时，将 `model_kwargs` 中的 `device` 设置为 `"xpu"` 将使嵌入模型运行在 Intel GPU 上，并受益于 IPEX-LLM 的优化：

```python
from langchain_community.embeddings import IpexLLMBgeEmbeddings

embedding_model = IpexLLMBgeEmbeddings(
    model_name="BAAI/bge-large-en-v1.5",
    model_kwargs={"device": "xpu"},
    encode_kwargs={"normalize_embeddings": True},
)
```

API 参考
- [IpexLLMBgeEmbeddings](https://api.python.langchain.com/en/latest/embeddings/langchain_community.embeddings.ipex_llm.IpexLLMBgeEmbeddings.html)


```python
sentence = "IPEX-LLM is a PyTorch library for running LLM on Intel CPU and GPU (e.g., local PC with iGPU, discrete GPU such as Arc, Flex and Max) with very low latency."
query = "What is IPEX-LLM?"

text_embeddings = embedding_model.embed_documents([sentence, query])
print(f"text_embeddings[0][:10]: {text_embeddings[0][:10]}")
print(f"text_embeddings[1][:10]: {text_embeddings[1][:10]}")

query_embedding = embedding_model.embed_query(query)
print(f"query_embedding[:10]: {query_embedding[:10]}")
```

## 相关

- 嵌入模型 [概念指南](/docs/concepts/#embedding-models)
- 嵌入模型 [操作指南](/docs/how_to/#embedding-models)