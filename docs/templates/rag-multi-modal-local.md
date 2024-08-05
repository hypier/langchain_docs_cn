# rag-multi-modal-local

视觉搜索对于许多使用iPhone或Android设备的人来说是一个熟悉的应用。它允许用户使用自然语言搜索照片。

随着开源多模态LLM的发布，现在可以为自己的私人照片库构建这种应用。

这个模板演示了如何在您的照片集合上执行私人视觉搜索和问答。

它使用[`nomic-embed-vision-v1`](https://huggingface.co/nomic-ai/nomic-embed-vision-v1)多模态嵌入来嵌入图像，并使用`Ollama`进行问答。

给定一个问题，相关照片将被检索并传递给您选择的开源多模态LLM进行答案合成。

## 输入

在 `/docs` 目录中提供一组照片。

默认情况下，此模板包含 3 张食物图片的玩具集合。

可以询问的示例问题包括：
```
我吃了什么样的软冰淇淋？
```

实际上，可以测试更大数量的图像。

要创建图像索引，请运行：
```
poetry install
python ingest.py
```

## 存储

此模板将使用 [nomic-embed-vision-v1](https://huggingface.co/nomic-ai/nomic-embed-vision-v1) 多模态嵌入来嵌入图像。

第一次运行应用程序时，它将自动下载多模态嵌入模型。

您可以在 `rag_chroma_multi_modal/ingest.py` 中选择其他模型，例如 `OpenCLIPEmbeddings`。
```
langchain_experimental.open_clip import OpenCLIPEmbeddings

embedding_function=OpenCLIPEmbeddings(
        model_name="ViT-H-14", checkpoint="laion2b_s32b_b79k"
        )

vectorstore_mmembd = Chroma(
    collection_name="multi-modal-rag",
    persist_directory=str(re_vectorstore_path),
    embedding_function=embedding_function
)
```

## LLM

此模板将使用 [Ollama](https://python.langchain.com/docs/integrations/chat/ollama#multi-modal)。

下载最新版本的 Ollama: https://ollama.ai/

拉取一个开源的多模态 LLM: 例如，https://ollama.ai/library/bakllava

```
ollama pull bakllava
```

该应用默认配置为 `bakllava`。但您可以在 `chain.py` 和 `ingest.py` 中更改以使用不同下载的模型。

## 用法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一包安装，您可以执行：

```shell
langchain app new my-app --package rag-chroma-multi-modal
```

如果您想将其添加到现有项目中，可以直接运行：

```shell
langchain app add rag-chroma-multi-modal
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_chroma_multi_modal import chain as rag_chroma_multi_modal_chain

add_routes(app, rag_chroma_multi_modal_chain, path="/rag-chroma-multi-modal")
```

（可选）现在让我们配置 LangSmith。 
LangSmith 将帮助我们跟踪、监控和调试 LangChain 应用程序。 
您可以在此处注册 LangSmith [here](https://smith.langchain.com/)。 
如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接通过以下方式启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行于 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/rag-chroma-multi-modal/playground](http://127.0.0.1:8000/rag-chroma-multi-modal/playground) 访问游乐场。  

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-chroma-multi-modal")
```