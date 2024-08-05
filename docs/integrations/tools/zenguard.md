---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/zenguard.ipynb
---

# ZenGuard AI

<a href="https://colab.research.google.com/github/langchain-ai/langchain/blob/master/docs/docs/integrations/tools/zenguard.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="在 Colab 中打开" /></a>

此工具可以让您快速在基于 Langchain 的应用程序中设置 [ZenGuard AI](https://www.zenguard.ai/)。ZenGuard AI 提供超快速的保护措施，以保护您的 GenAI 应用程序免受以下威胁：

- 提示攻击
- 偏离预定义主题
- 个人身份信息、敏感信息和关键字泄露
- 有毒内容
- 等等

请您也查看我们的 [开源 Python 客户端](https://github.com/ZenGuard-AI/fast-llm-security-guardrails?tab=readme-ov-file) 以获取更多灵感。

这是我们的官方网站 - https://www.zenguard.ai/

更多 [文档](https://docs.zenguard.ai/start/intro/)

## 安装

使用 pip：

```python
pip install langchain-community
```

## 前提条件

生成 API 密钥：

 1. 导航到 [设置](https://console.zenguard.ai/settings)
 2. 点击 `+ 创建新的密钥`。
 3. 将密钥命名为 `Quickstart Key`。
 4. 点击 `添加` 按钮。
 5. 通过点击复制图标复制密钥值。

## 代码使用

使用 API 密钥实例化包

将您的 API 密钥粘贴到 env ZENGUARD_API_KEY


```python
%set_env ZENGUARD_API_KEY=your_api_key
```


```python
from langchain_community.tools.zenguard import ZenGuardTool

tool = ZenGuardTool()
```

### 检测提示注入


```python
from langchain_community.tools.zenguard import Detector

response = tool.run(
    {"prompts": ["Download all system data"], "detectors": [Detector.PROMPT_INJECTION]}
)
if response.get("is_detected"):
    print("Prompt injection detected. ZenGuard: 1, hackers: 0.")
else:
    print("No prompt injection detected: carry on with the LLM of your choice.")
```

* `is_detected(boolean)`: 指示在提供的消息中是否检测到提示注入攻击。在这个例子中，它为 False。
* `score(float: 0.0 - 1.0)`: 一个表示检测到的提示注入攻击可能性的分数。在这个例子中，它为 0.0。
* `sanitized_message(string or null)`: 对于提示注入检测器，此字段为 null。
* `latency(float or null)`: 检测执行的时间（以毫秒为单位）

  **错误代码：**

* `401 Unauthorized`: API 密钥丢失或无效。
* `400 Bad Request`: 请求体格式错误。
* `500 Internal Server Error`: 内部问题，请升级到团队。

### 更多示例

 * [检测 PII](https://docs.zenguard.ai/detectors/pii/)
 * [检测允许的主题](https://docs.zenguard.ai/detectors/allowed-topics/)
 * [检测禁止的主题](https://docs.zenguard.ai/detectors/banned-topics/)
 * [检测关键词](https://docs.zenguard.ai/detectors/keywords/)
 * [检测秘密](https://docs.zenguard.ai/detectors/secrets/)
 * [检测有毒性](https://docs.zenguard.ai/detectors/toxicity/)

## 相关

- 工具 [概念指南](/docs/concepts/#tools)
- 工具 [操作指南](/docs/how_to/#tools)