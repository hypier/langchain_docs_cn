# rag-redis-multi-modal-multi-vector

多模态 LLM 使得能够进行图像问答的视觉助手成为可能。

此模板创建了一个用于幻灯片的视觉助手，幻灯片通常包含图表或图形等视觉元素。

它使用 GPT-4V 为每个幻灯片创建图像摘要，嵌入摘要并将其存储在 Redis 中。

在给定问题的情况下，相关的幻灯片被检索并传递给 GPT-4V 进行答案合成。

## 输入

在 `/docs` 目录中提供一个 PDF 格式的幻灯片文档。

默认情况下，此模板包含有关 NVIDIA 最近收益的幻灯片文档。

可以提出的示例问题包括：
```
1/ H100 TensorRT 可以提高 LLama2 推理性能多少？
2/ 从 2020 年到 2023 年，GPU 加速应用程序的百分比变化是多少？
```

要创建幻灯片文档的索引，请运行：
```
poetry install
poetry shell
python ingest.py
```

## 存储

这里是模板将用于创建幻灯片索引的过程（见 [blog](https://blog.langchain.dev/multi-modal-rag-template/)）：

* 将幻灯片提取为图像集合
* 使用 GPT-4V 对每个图像进行摘要
* 使用文本嵌入将图像摘要嵌入，并链接到原始图像
* 根据图像摘要与用户输入问题之间的相似性检索相关图像
* 将这些图像传递给 GPT-4V 进行答案合成

### Redis
此模板使用 [Redis](https://redis.com) 来支持 [MultiVectorRetriever](https://python.langchain.com/docs/modules/data_connection/retrievers/multi_vector)，包括：
- Redis 作为 [VectorStore](https://python.langchain.com/docs/integrations/vectorstores/redis)（用于存储 + 索引图像摘要嵌入）
- Redis 作为 [ByteStore](https://python.langchain.com/docs/integrations/stores/redis)（用于存储图像）

确保在 [云端](https://redis.com/try-free)（免费）或使用 [docker](https://redis.io/docs/install/install-stack/docker/) 在本地部署一个 Redis 实例。

这将为您提供一个可访问的 Redis 端点，您可以将其用作 URL。如果在本地部署，只需使用 `redis://localhost:6379`。

## LLM

该应用将根据文本输入与图像摘要（文本）之间的相似性检索图像，并将图像传递给GPT-4V进行答案合成。

## 环境设置

设置 `OPENAI_API_KEY` 环境变量以访问 OpenAI GPT-4V。

设置 `REDIS_URL` 环境变量以访问您的 Redis 数据库。

## 使用方法

要使用此软件包，您首先需要安装 LangChain CLI：

```shell
pip install -U langchain-cli
```

要创建一个新的 LangChain 项目并将其作为唯一的软件包安装，您可以执行：

```shell
langchain app new my-app --package rag-redis-multi-modal-multi-vector
```

如果您想将其添加到现有项目中，只需运行：

```shell
langchain app add rag-redis-multi-modal-multi-vector
```

并将以下代码添加到您的 `server.py` 文件中：
```python
from rag_redis_multi_modal_multi_vector import chain as rag_redis_multi_modal_chain_mv

add_routes(app, rag_redis_multi_modal_chain_mv, path="/rag-redis-multi-modal-multi-vector")
```

（可选）现在我们来配置 LangSmith。 
LangSmith 将帮助我们跟踪、监视和调试 LangChain 应用程序。 
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

这将启动 FastAPI 应用程序，服务器在本地运行于 
[http://localhost:8000](http://localhost:8000)

我们可以在 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 查看所有模板
我们可以在 [http://127.0.0.1:8000/rag-redis-multi-modal-multi-vector/playground](http://127.0.0.1:8000/rag-redis-multi-modal-multi-vector/playground) 访问游乐场  

我们可以通过以下代码访问模板：

```python
from langserve.client import RemoteRunnable

runnable = RemoteRunnable("http://localhost:8000/rag-redis-multi-modal-multi-vector")
```