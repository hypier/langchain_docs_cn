---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/how_to/chat_model_rate_limiting.ipynb
---

# 如何处理速率限制

:::info 前提条件

本指南假设您熟悉以下概念：
- [聊天模型](/docs/concepts/#chat-models)
- [大型语言模型 (LLMs)](/docs/concepts/#llms)
:::


您可能会遇到由于请求过多而被模型提供者 API 限制速率的情况。

例如，如果您在测试数据集上运行多个并行查询以基准测试聊天模型，就可能会发生这种情况。

如果您面临这样的情况，可以使用速率限制器来帮助将您的请求速率与 API 允许的速率相匹配。

:::info 需要 ``langchain-core >= 0.2.24``

此功能是在 ``langchain-core == 0.2.24`` 中添加的。请确保您的软件包是最新的。
:::

## 初始化速率限制器

Langchain 提供了内置的内存速率限制器。该速率限制器是线程安全的，可以被同一进程中的多个线程共享。

提供的速率限制器只能限制单位时间内的请求数量。如果您还需要根据请求的大小进行限制，它将无能为力。


```python
from langchain_core.rate_limiters import InMemoryRateLimiter

rate_limiter = InMemoryRateLimiter(
    requests_per_second=0.1,  # <-- 超级慢！我们每 10 秒只能发出一次请求！！
    check_every_n_seconds=0.1,  # 每 100 毫秒醒来一次，检查是否允许发出请求，
    max_bucket_size=10,  # 控制最大突发大小。
)
```

## 选择模型

选择任意模型，并通过 `rate_limiter` 属性传递速率限制器。

```python
import os
import time
from getpass import getpass

if "ANTHROPIC_API_KEY" not in os.environ:
    os.environ["ANTHROPIC_API_KEY"] = getpass()


from langchain_anthropic import ChatAnthropic

model = ChatAnthropic(model_name="claude-3-opus-20240229", rate_limiter=rate_limiter)
```

让我们确认速率限制器的工作情况。我们应该只能每 10 秒调用一次模型。

```python
for _ in range(5):
    tic = time.time()
    model.invoke("hello")
    toc = time.time()
    print(toc - tic)
```
```output
11.599073648452759
10.7502121925354
10.244257926940918
8.83088755607605
11.645203590393066
```