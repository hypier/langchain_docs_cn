# 烟花

本页介绍如何在 Langchain 中使用 [Fireworks](https://fireworks.ai/) 模型。

## 安装与设置

- 安装 Fireworks 集成包。

  ```
  pip install langchain-fireworks
  ```

- 通过在 [fireworks.ai](https://fireworks.ai) 注册获取 Fireworks API 密钥。
- 通过设置 FIREWORKS_API_KEY 环境变量进行身份验证。

## 身份验证

使用您的 Fireworks API 密钥进行身份验证有两种方法：

1.  设置 `FIREWORKS_API_KEY` 环境变量。

    ```python
    os.environ["FIREWORKS_API_KEY"] = "<KEY>"
    ```

2.  在 Fireworks LLM 模块中设置 `api_key` 字段。

    ```python
    llm = Fireworks(api_key="<KEY>")
    ```

## 使用 Fireworks LLM 模块

Fireworks 通过 LLM 模块与 Langchain 集成。在这个例子中，我们将使用 mixtral-8x7b-instruct 模型。

```python
from langchain_fireworks import Fireworks 

llm = Fireworks(
    api_key="<KEY>",
    model="accounts/fireworks/models/mixtral-8x7b-instruct",
    max_tokens=256)
llm("Name 3 sports.")
```

有关更详细的操作指南，请参见 [这里](/docs/integrations/llms/Fireworks)。