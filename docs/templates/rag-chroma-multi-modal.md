# rag-chroma-multi-modal

多模态 LLM 使得视觉助手能够对图像进行问答。

此模板创建一个用于幻灯片的视觉助手，幻灯片通常包含图表或图形等视觉元素。

它使用 OpenCLIP 嵌入将所有幻灯片图像嵌入并存储在 Chroma 中。

给定一个问题，相关幻灯片会被检索并传递给 GPT-4V 进行答案合成。

## 输入

在 `/docs` 目录中提供一个 PDF 格式的幻灯片文档。

默认情况下，此模板包含有关 DataDog（一家公共科技公司）第三季度收益的幻灯片文档。

可以询问的示例问题包括：
```
Datadog 有多少客户？
Datadog 平台在 FY20、FY21 和 FY22 的同比增长百分比是多少？
```

要创建幻灯片文档的索引，请运行：
```
poetry install
python ingest.py
```

## 存储

该模板将使用 [OpenCLIP](https://github.com/mlfoundations/open_clip) 多模态嵌入来嵌入图像。

您可以选择不同的嵌入模型选项（请参见结果 [这里](https://github.com/mlfoundations/open_clip/blob/main/docs/openclip_results.csv)）。

第一次运行应用程序时，它将自动下载多模态嵌入模型。

默认情况下，LangChain 将使用具有中等性能但内存需求较低的嵌入模型 `ViT-H-14`。

您可以在 `rag_chroma_multi_modal/ingest.py` 中选择其他 `OpenCLIPEmbeddings` 模型：
```
vectorstore_mmembd = Chroma(
    collection_name="multi-modal-rag",
    persist_directory=str(re_vectorstore_path),
    embedding_function=OpenCLIPEmbeddings(
        model_name="ViT-H-14", checkpoint="laion2b_s32b_b79k"
    ),
)
```

## LLM

该应用将根据文本输入与图像之间的相似性检索图像，这两者都映射到多模态嵌入空间。然后，它将把图像传递给 GPT-4V。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI GPT-4V.

## 使用方法

要使用这个包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package rag-chroma-multi-modal
```

如果您想将其添加到现有项目中，只需运行：

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
您可以在 [这里](https://smith.langchain.com/) 注册 LangSmith。 
如果您没有访问权限，可以跳过此部分。

```shell
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=<your-api-key>
export LANGCHAIN_PROJECT=<your-project>  # 如果未指定，默认为 "default"
```

如果您在此目录中，则可以直接通过以下命令启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/rag-chroma-multi-modal/playground](http://127.0.0.1:8000/rag-chroma-multi-modal/playground) 访问游乐场。

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-chroma-multi-modal")
```