---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/toolkits/requests.ipynb
sidebar_label: 请求
---

# Requests Toolkit

我们可以使用 Requests [toolkit](/docs/concepts/#toolkits) 来构建生成 HTTP 请求的代理。

有关所有 API 工具包功能和配置的详细文档，请访问 [RequestsToolkit](https://api.python.langchain.com/en/latest/agent_toolkits/langchain_community.agent_toolkits.openapi.toolkit.RequestsToolkit.html) 的 API 参考。

## ⚠️ 安全提示 ⚠️
在赋予模型执行现实世界行为的自由裁量权时，存在固有风险。采取预防措施以减轻这些风险：

- 确保与工具相关的权限范围狭窄（例如，用于数据库操作或API请求）；
- 在需要时，利用人机协作的工作流程。

## 设置

### 安装

该工具包位于 `langchain-community` 包中：


```python
%pip install -qU langchain-community
```

请注意，如果您希望从单个工具的运行中获得自动跟踪，您还可以通过取消注释下面的内容来设置您的 [LangSmith](https://docs.smith.langchain.com/) API 密钥：


```python
# os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
# os.environ["LANGSMITH_TRACING"] = "true"
```

## 实例化

首先，我们将演示一个最小示例。

**注意**：赋予模型执行现实世界操作的自由裁量权固有风险。我们必须通过设置 `allow_dangerous_request=True` 来“选择接受”这些风险，以使用这些工具。
**这可能会导致调用不必要的请求**。请确保您的自定义 OpenAPI 规范 (yaml) 是安全的，并且与工具相关的权限是狭义的。

```python
ALLOW_DANGEROUS_REQUEST = True
```

我们可以使用 [JSONPlaceholder](https://jsonplaceholder.typicode.com) API 作为测试平台。

让我们创建（其 API 规范的一个子集）：

```python
from typing import Any, Dict, Union

import requests
import yaml


def _get_schema(response_json: Union[dict, list]) -> dict:
    if isinstance(response_json, list):
        response_json = response_json[0] if response_json else {}
    return {key: type(value).__name__ for key, value in response_json.items()}


def _get_api_spec() -> str:
    base_url = "https://jsonplaceholder.typicode.com"
    endpoints = [
        "/posts",
        "/comments",
    ]
    common_query_parameters = [
        {
            "name": "_limit",
            "in": "query",
            "required": False,
            "schema": {"type": "integer", "example": 2},
            "description": "限制结果数量",
        }
    ]
    openapi_spec: Dict[str, Any] = {
        "openapi": "3.0.0",
        "info": {"title": "JSONPlaceholder API", "version": "1.0.0"},
        "servers": [{"url": base_url}],
        "paths": {},
    }
    # 遍历端点以构建路径
    for endpoint in endpoints:
        response = requests.get(base_url + endpoint)
        if response.status_code == 200:
            schema = _get_schema(response.json())
            openapi_spec["paths"][endpoint] = {
                "get": {
                    "summary": f"获取 {endpoint[1:]}",
                    "parameters": common_query_parameters,
                    "responses": {
                        "200": {
                            "description": "成功响应",
                            "content": {
                                "application/json": {
                                    "schema": {"type": "object", "properties": schema}
                                }
                            },
                        }
                    },
                }
            }
    return yaml.dump(openapi_spec, sort_keys=False)


api_spec = _get_api_spec()
```

接下来，我们可以实例化工具包。我们不需要为此 API 提供授权或其他标头：

```python
from langchain_community.agent_toolkits.openapi.toolkit import RequestsToolkit
from langchain_community.utilities.requests import TextRequestsWrapper

toolkit = RequestsToolkit(
    requests_wrapper=TextRequestsWrapper(headers={}),
    allow_dangerous_requests=ALLOW_DANGEROUS_REQUEST,
)
```

## 工具

查看可用工具：


```python
tools = toolkit.get_tools()

tools
```



```output
[RequestsGetTool(requests_wrapper=TextRequestsWrapper(headers={}, aiosession=None, auth=None, response_content_type='text', verify=True), allow_dangerous_requests=True),
 RequestsPostTool(requests_wrapper=TextRequestsWrapper(headers={}, aiosession=None, auth=None, response_content_type='text', verify=True), allow_dangerous_requests=True),
 RequestsPatchTool(requests_wrapper=TextRequestsWrapper(headers={}, aiosession=None, auth=None, response_content_type='text', verify=True), allow_dangerous_requests=True),
 RequestsPutTool(requests_wrapper=TextRequestsWrapper(headers={}, aiosession=None, auth=None, response_content_type='text', verify=True), allow_dangerous_requests=True),
 RequestsDeleteTool(requests_wrapper=TextRequestsWrapper(headers={}, aiosession=None, auth=None, response_content_type='text', verify=True), allow_dangerous_requests=True)]
```


- [RequestsGetTool](https://api.python.langchain.com/en/latest/tools/langchain_community.tools.requests.tool.RequestsGetTool.html)
- [RequestsPostTool](https://api.python.langchain.com/en/latest/tools/langchain_community.tools.requests.tool.RequestsPostTool.html)
- [RequestsPatchTool](https://api.python.langchain.com/en/latest/tools/langchain_community.tools.requests.tool.RequestsPatchTool.html)
- [RequestsPutTool](https://api.python.langchain.com/en/latest/tools/langchain_community.tools.requests.tool.RequestsPutTool.html)
- [RequestsDeleteTool](https://api.python.langchain.com/en/latest/tools/langchain_community.tools.requests.tool.RequestsDeleteTool.html)

## 在代理中使用


```python
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import create_react_agent

llm = ChatOpenAI(model="gpt-3.5-turbo-0125")

system_message = """
You have access to an API to help answer user queries.
Here is documentation on the API:
{api_spec}
""".format(api_spec=api_spec)

agent_executor = create_react_agent(llm, tools, state_modifier=system_message)
```


```python
example_query = "Fetch the top two posts. What are their titles?"

events = agent_executor.stream(
    {"messages": [("user", example_query)]},
    stream_mode="values",
)
for event in events:
    event["messages"][-1].pretty_print()
```
```output
================================[1m 人类消息 [0m=================================

Fetch the top two posts. What are their titles?
==================================[1m AI 消息 [0m==================================
工具调用:
  requests_get (call_RV2SOyzCnV5h2sm4WPgG8fND)
 调用 ID: call_RV2SOyzCnV5h2sm4WPgG8fND
  参数:
    url: https://jsonplaceholder.typicode.com/posts?_limit=2
=================================[1m 工具消息 [0m=================================
名称: requests_get

[
  {
    "userId": 1,
    "id": 1,
    "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
    "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
  },
  {
    "userId": 1,
    "id": 2,
    "title": "qui est esse",
    "body": "est rerum tempore vitae\nsequi sint nihil reprehenderit dolor beatae ea dolores neque\nfugiat blanditiis voluptate porro vel nihil molestiae ut reiciendis\nqui aperiam non debitis possimus qui neque nisi nulla"
  }
]
==================================[1m AI 消息 [0m==================================

The titles of the top two posts are:
1. "sunt aut facere repellat provident occaecati excepturi optio reprehenderit"
2. "qui est esse"
```

## API 参考

有关所有 API 工具包功能和配置的详细文档，请访问 [RequestsToolkit](https://api.python.langchain.com/en/latest/agent_toolkits/langchain_community.agent_toolkits.openapi.toolkit.RequestsToolkit.html) 的 API 参考。