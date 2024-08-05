# Portkey

[Portkey](https://portkey.ai) 是 AI 应用的控制面板。凭借其流行的 AI 网关和可观察性套件，数百个团队交付 **可靠**、**成本效益高** 和 **快速** 的应用程序。

## LLMOps for Langchain

Portkey 为 Langchain 带来了生产就绪性。使用 Portkey，您可以 
- [x] 通过统一的 API 连接 150 多个模型，
- [x] 查看 42 多个 **指标和日志** 来跟踪所有请求，
- [x] 启用 **语义缓存** 以减少延迟和成本，
- [x] 实现自动 **重试和回退** 以处理失败的请求，
- [x] 为请求添加 **自定义标签** 以便更好地跟踪和分析，以及 [更多](https://portkey.ai/docs)。

## 快速入门 - Portkey 和 Langchain
由于 Portkey 完全兼容 OpenAI 签名，您可以通过 `ChatOpenAI` 接口连接到 Portkey AI 网关。

- 将 `base_url` 设置为 `PORTKEY_GATEWAY_URL`
- 使用 `createHeaders` 辅助方法添加 `default_headers` 以使用 Portkey 需要的头信息。

首先，通过 [在这里注册](https://app.portkey.ai/signup) 获取您的 Portkey API 密钥。 （点击左下角的个人资料图标，然后点击“复制 API 密钥”）或在 [您自己的环境中](https://github.com/Portkey-AI/gateway/blob/main/docs/installation-deployments.md) 部署开源 AI 网关。

接下来，安装 Portkey SDK
```python
pip install -U portkey_ai
```

现在，我们可以通过更新 Langchain 中的 `ChatOpenAI` 模型来连接到 Portkey AI 网关
```python
from langchain_openai import ChatOpenAI
from portkey_ai import createHeaders, PORTKEY_GATEWAY_URL

PORTKEY_API_KEY = "..." # Not needed when hosting your own gateway
PROVIDER_API_KEY = "..." # Add the API key of the AI provider being used 

portkey_headers = createHeaders(api_key=PORTKEY_API_KEY,provider="openai")

llm = ChatOpenAI(api_key=PROVIDER_API_KEY, base_url=PORTKEY_GATEWAY_URL, default_headers=portkey_headers)

llm.invoke("What is the meaning of life, universe and everything?")
```

请求通过您的 Portkey AI 网关路由到指定的 `provider`。 Portkey 还将开始记录您帐户中的所有请求，使调试变得非常简单。

![从 Langchain 查看 Portkey 的日志](https://assets.portkey.ai/docs/langchain-logs.gif)

## 通过 AI Gateway 使用 150+ 模型
AI gateway 的强大之处在于您能够使用上述代码片段连接超过 150 个模型，涵盖 20 多个通过 AI gateway 支持的提供者。

让我们修改上述代码，以调用 Anthropic 的 `claude-3-opus-20240229` 模型。

Portkey 支持 **[虚拟密钥](https://docs.portkey.ai/docs/product/ai-gateway-streamline-llm-integrations/virtual-keys)**，这是一种安全存储和管理 API 密钥的简便方法。让我们尝试使用虚拟密钥进行 LLM 调用。您可以在 Portkey 中导航到虚拟密钥标签，并为 Anthropic 创建一个新密钥。

`virtual_key` 参数设置所使用的 AI 提供者的身份验证和提供者。在我们的例子中，我们使用的是 Anthropic 的虚拟密钥。

> 请注意，`api_key` 可以留空，因为该身份验证将不会被使用。

```python
from langchain_openai import ChatOpenAI
from portkey_ai import createHeaders, PORTKEY_GATEWAY_URL

PORTKEY_API_KEY = "..."
VIRTUAL_KEY = "..." # Anthropic's virtual key we copied above

portkey_headers = createHeaders(api_key=PORTKEY_API_KEY,virtual_key=VIRTUAL_KEY)

llm = ChatOpenAI(api_key="X", base_url=PORTKEY_GATEWAY_URL, default_headers=portkey_headers, model="claude-3-opus-20240229")

llm.invoke("What is the meaning of life, universe and everything?")
```

Portkey AI gateway 将对 Anthropic 的 API 请求进行身份验证，并以 OpenAI 格式返回响应，以供您使用。

AI gateway 扩展了 Langchain 的 `ChatOpenAI` 类，使其成为调用任何提供者和任何模型的单一接口。

## 高级路由 - 负载均衡、回退、重试  
Portkey AI Gateway 通过以配置为先的方法，为 Langchain 带来了负载均衡、回退、实验和金丝雀测试等功能。

让我们举一个 **例子**，假设我们想将流量在 `gpt-4` 和 `claude-opus` 之间以 50:50 的比例进行拆分，以测试这两个大型模型。网关的配置如下所示：

```python
config = {
    "strategy": {
         "mode": "loadbalance"
    },
    "targets": [{
        "virtual_key": "openai-25654", # OpenAI's virtual key
        "override_params": {"model": "gpt4"},
        "weight": 0.5
    }, {
        "virtual_key": "anthropic-25654", # Anthropic's virtual key
        "override_params": {"model": "claude-3-opus-20240229"},
        "weight": 0.5
    }]
}
```

然后，我们可以在从 langchain 发出的请求中使用此配置。

```python
portkey_headers = createHeaders(
    api_key=PORTKEY_API_KEY,
    config=config
)

llm = ChatOpenAI(api_key="X", base_url=PORTKEY_GATEWAY_URL, default_headers=portkey_headers)

llm.invoke("What is the meaning of life, universe and everything?")
```

当调用 LLM 时，Portkey 将根据定义的权重比例将请求分配给 `gpt-4` 和 `claude-3-opus-20240229`。

您可以在 [这里](https://docs.portkey.ai/docs/api-reference/config-object#examples) 找到更多配置示例。

## **追踪链和代理**

Portkey 的 Langchain 集成使您能够全面了解代理的运行情况。让我们以一个 [流行的代理工作流程](https://python.langchain.com/docs/use_cases/tool_use/quickstart/#agents) 为例。

我们只需要修改 `ChatOpenAI` 类以使用上述 AI Gateway。

```python
from langchain import hub  
from langchain.agents import AgentExecutor, create_openai_tools_agent  
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
from portkey_ai import PORTKEY_GATEWAY_URL, createHeaders
 
prompt = hub.pull("hwchase17/openai-tools-agent")

portkey_headers = createHeaders(
    api_key=PORTKEY_API_KEY,
    virtual_key=OPENAI_VIRTUAL_KEY,
    trace_id="uuid-uuid-uuid-uuid"
)

@tool
def multiply(first_int: int, second_int: int) -> int:
    """将两个整数相乘。"""
    return first_int * second_int
  
  
@tool  
def exponentiate(base: int, exponent: int) -> int:  
    "将底数提高到指数的幂。"  
    return base**exponent  
  
  
tools = [multiply, exponentiate]

model = ChatOpenAI(api_key="X", base_url=PORTKEY_GATEWAY_URL, default_headers=portkey_headers, temperature=0)
  
# 构建 OpenAI Tools 代理  
agent = create_openai_tools_agent(model, tools, prompt)

# 通过传入代理和工具创建代理执行器
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

agent_executor.invoke({
    "input": "将 3 的五次方乘以 36，然后对结果进行平方"
})
```

**您可以在 Portkey 仪表板上查看请求日志及其追踪 ID：**
![Langchain 代理日志在 Portkey](https://assets.portkey.ai/docs/agent_tracing.gif)

附加文档可在此处找到：
- 可观察性 - https://portkey.ai/docs/product/observability-modern-monitoring-for-llms
- AI Gateway - https://portkey.ai/docs/product/ai-gateway-streamline-llm-integrations
- 提示库 - https://portkey.ai/docs/product/prompt-library

您可以在这里查看我们流行的开源 AI Gateway - https://github.com/portkey-ai/gateway

有关每个功能及其使用的详细信息，请 [参考 Portkey 文档](https://portkey.ai/docs)。如果您有任何问题或需要进一步的帮助，请 [通过 Twitter 联系我们。](https://twitter.com/portkeyai) 或我们的 [支持邮箱](mailto:hello@portkey.ai)。