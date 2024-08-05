# rag-multi-modal-mv-local

视觉搜索是许多iPhone或Android设备用户熟悉的应用。它允许用户使用自然语言搜索照片。

随着开源多模态LLM的发布，您可以为自己的私人照片集合构建这种应用程序。

此模板演示如何在您的照片集合上执行私人视觉搜索和问答。

它使用您选择的开源多模态LLM为每张照片创建图像摘要，嵌入摘要并将其存储在Chroma中。

给定一个问题，相关照片将被检索并传递给多模态LLM进行答案合成。

## 输入

在 `/docs` 目录中提供一组照片。

默认情况下，此模板包含 3 张食物照片的玩具集合。

该应用程序将根据提供的关键词或问题查找并总结照片：
```
我吃了什么样的冰淇淋？
```

实际上，可以测试更大的图像语料库。

要创建图像索引，请运行：
```
poetry install
python ingest.py
```

## 存储

这里是模板用于创建幻灯片索引的过程（参见 [blog](https://blog.langchain.dev/multi-modal-rag-template/)）：

* 给定一组图像
* 它使用本地多模态 LLM ([bakllava](https://ollama.ai/library/bakllava)) 来总结每个图像
* 将图像摘要嵌入并链接到原始图像
* 给定用户问题，它将根据图像摘要与用户输入之间的相似性（使用 Ollama 嵌入）找到相关图像
* 它将把这些图像传递给 bakllava 进行答案合成

默认情况下，这将使用 [LocalFileStore](https://python.langchain.com/docs/integrations/stores/file_system) 来存储图像，并使用 Chroma 来存储摘要。

## LLM 和嵌入模型

我们将使用 [Ollama](https://python.langchain.com/docs/integrations/chat/ollama#multi-modal) 来生成图像摘要、嵌入和最终的图像问答。

下载最新版本的 Ollama: https://ollama.ai/

拉取一个开源多模态 LLM: 例如，https://ollama.ai/library/bakllava

拉取一个开源嵌入模型: 例如，https://ollama.ai/library/llama2:7b

```
ollama pull bakllava
ollama pull llama2:7b
```

该应用程序默认配置为 `bakllava`。但您可以在 `chain.py` 和 `ingest.py` 中更改以使用不同的下载模型。

该应用程序将根据文本输入与图像摘要之间的相似性检索图像，并将图像传递给 `bakllava`。

## 使用方法

要使用此包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的包安装，您可以执行：

```shell
langchain app new my-app --package rag-multi-modal-mv-local
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-multi-modal-mv-local
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_multi_modal_mv_local import chain as rag_multi_modal_mv_local_chain

add_routes(app, rag_multi_modal_mv_local_chain, path="/rag-multi-modal-mv-local")
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

如果您在此目录内，则可以直接通过以下命令启动 LangServe 实例：

```shell
langchain serve
```

这将启动 FastAPI 应用程序，服务器在本地运行，地址为 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板。
我们可以在 [http://127.0.0.1:8000/rag-multi-modal-mv-local/playground](http://127.0.0.1:8000/rag-multi-modal-mv-local/playground) 访问游乐场。

我们可以通过代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-multi-modal-mv-local")
```